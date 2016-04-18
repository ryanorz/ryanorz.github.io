---
layout:     post
title:      "Using Linux namespaces to isolate network in a new environment"
subtitle:   "clone() is more powerful than fork()"
date:       2016-04-18
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - Programming
    - linux
--- 

# Introduction

Namespaces are a Linux kernel feature that isolates and virtualizes resources (PID, hostname, userid, network, ipc, filesystem) of a collection of processes.

`Docker` uses the machinism to implement environment isolation.

If you want to know the namespaces basic introduction, please `man namespaces`, `man clone` and etc..

# Fork vs. clone

To what `fork()` can do, `clone()` can do them, too. Besides, `clone()` applies more feature support. 
`clone()` allows the child process to share parts of its execution context with the calling process, such as the memory space, the table of file  descriptors,  and the table of signal handlers.
`clone()` can also isolate the resources from its' parent by using namespaces.

# a sample of fork

```c++
/**
 * @brief just like system() and execv(), execute a command and get return value and output
 * @note  argument do not need to be escaped. system() argments need to be escaped, which is really a disgusting thing.
 * @param args argments list, args[0] is execute absolute path
 * @param output standard output
 * @return int command return value
 */
int execv_ext(const vector<string> &args, string &output)
{
	int pipefd[2];
	pipe(pipefd);

	pid_t pid = fork();
	if (pid == -1) { // fork error
		syslog(LOG_ERR, "exec_args fork error.");
		abort();
	} else if (pid == 0) { // child
		if (args.size() == 0)
			return -1;

		close(pipefd[0]);      // close reading end in the child
		dup2(pipefd[1], 1);    // send stdout to the pipe
		//dup2(pipefd[1], 2);  // send stderr to the pipe
		close(pipefd[1]);      // this descriptor is no longer needed

		char *argv[args.size() + 1];
		argv[args.size()] = NULL;
		for (size_t i = 0; i < args.size(); ++i)
			argv[i] = const_cast<char *>(args[i].c_str());
		argv[args.size()] = NULL;
		if (args[0].empty() || !exists(args[0])) {
			syslog(LOG_WARNING, "command %s not found.", args[0].c_str());
			exit(-1);
		}
		execv(argv[0], argv);
		syslog(LOG_ERR, "%s", strerror(errno));
		abort();
	} else { // parent
		close(pipefd[1]);    // close the write end of the pipe in the parent
		char buffer[1024];
		ssize_t len;
		while((len = read(pipefd[0], buffer, 1024)) > 0)
			output += string(buffer, len);
		close(pipefd[0]);    // close the reading end of the pipe in the parent
		int stat_val;
		wait(&stat_val);
		if (WIFEXITED(stat_val))
			return WEXITSTATUS(stat_val);
		else
			return -1;
	}

	return -1;
}
```

# a sample of clone to isolate network and socket communicate between server and client

Because my project must execute command in multi-threads program. Because fork in multi-threads program may be very dangerous, I have to write a service (daemon) to deal with exec responses. I use socket communicate between server (for accept command, execute command and send the result and output to client) and client (send command to server and receive the result and output from server). I must impletment multi-clients run their command at the same time. And for system security, I don't want the command use network, because some command is unrealiable, it may use network do some bad things.

Then between server and client, I use json string to transfer their struct information. The server accept every client response and and deal the business logic things in a new process. After the child process over, the sever receive the result and output and then send to the client.

I change the execute command to send a message to short the code. It's just a struct. The code struct is below, it mainly shows the communication between them.

```c++
/**
 * Client code
 * ignore header files
 */
static const char *socketfile = "/tmp/server_socket";

int main()
{
	// Set up the variables
	int sockfd;
	socklen_t len;
	struct sockaddr_un address;
	int result;

	// Create a socket for the client
	sockfd = socket(AF_UNIX, SOCK_STREAM | SOCK_CLOEXEC, 0);

	// Name the socket as agreed with the server
	address.sun_family = AF_UNIX;
	strcpy(address.sun_path, socketfile);
	len = sizeof(address);

	// Connect your socket to the server’s socket
	result = connect(sockfd, (struct sockaddr *)&address, len);
	if(result == -1) {
		syslog(LOG_ERR, "support exec client connect error. %m");
		printf("support exec client connect error.\n");
		abort();
	}
	
	const char *client_message = "THIS IS MESSAGE FROM CLIENT.\n";
	int message_len = strlen(client_message);
	int write_len = write(sockfd, client_message, message_len);
	if (write_len != message_len) {
		syslog(LOG_ERR, "client write message error. %m");
		close(sockfd);
		return -1;
	}
	printf("send message success.\n");
	char buffer[1024] = {0};
	ssize_t read_len = read(sockfd, buffer, 1024);
	if (read_len == -1) {
		syslog(LOG_ERR, "client read message error. %m");
		close(sockfd);
		return -1;
	}
	printf("Client receive : %s\n", buffer);
	close(sockfd);
	exit(0);
}
```


```c++
/**
 * Server code
 * ignore header files
 */

#define STACK_SIZE (1024 * 1024)
static const char *socketfile = "/tmp/server_socket";
static char stack[STACK_SIZE];

struct ConnectionInfo {
	pid_t child_pid;
	int   client_sockfd;
	int   pipefd[2];
	string cmd;
};

static unordered_map<int, ConnectionInfo> connection_table;  // hash table to save connection

static int is_close = 0;

static void wait_child(__attribute__((unused)) int sig, siginfo_t *siginfo, __attribute__((unused))void *unused)
{
	int status;
	wait(&status);
	unordered_map<int, ConnectionInfo>::iterator connectionIter;
	ConnectionInfo connection;
	connection.client_sockfd = -1;
	string output;
	connectionIter = connection_table.find(siginfo->si_pid);
	if (connectionIter != connection_table.end()) {
		connection = connectionIter->second;
		connection_table.erase(connectionIter);
		char buffer[1024];
		ssize_t read_len = read(connection.pipefd[0], &buffer, 1024);
		if (read_len == -1) {
			syslog(LOG_ERR, "server read error. %m");
		} else {
			output = string(buffer, read_len);
		}
		close(connection.pipefd[0]);
	} else  {
		syslog(LOG_ERR, "cannot find connection from hash table.");
	}
	if (connection.client_sockfd != -1) {
		int output_size = output.size();
		int write_len = write(connection.client_sockfd, output.c_str(), output_size);
		if (write_len != output_size)
			syslog(LOG_ERR, "server write output error. %m");
		close(connection.client_sockfd);
	}
}

static int deal_connection(void *_connection)
{
	prctl(PR_SET_PDEATHSIG, SIGTERM);

	system("ip addr");

	ConnectionInfo *connection = (ConnectionInfo *)_connection;
	if (connection->pipefd[0] < 0 || connection->pipefd[1] < 0) {
		syslog(LOG_ERR, "PIPE ERROR ??? %m");
	}
	close(connection->pipefd[0]);            // close reading end in the child
	if (connection->client_sockfd < 0) {
		syslog(LOG_ERR, "client_sockfd < 0 ??? %m");
		close(connection->pipefd[1]);    // this descriptor is no longer needed
		exit(-1);
	}
	cout << "DEAL_CONNECTION : cmd = " << connection->cmd << endl;
	// exec bla bla bla
	const char *server_message = "THIS IS SERVER MESSAGE .^_^.\n";
	int message_len = strlen(server_message);
	int write_len = write(connection->pipefd[1], server_message, message_len);
	if (write_len != message_len) {
		syslog(LOG_ERR, "child process write pipe error. %m");
		close(connection->pipefd[1]);    // this descriptor is no longer needed
		exit(-1);
	}
	exit(0);
}

static void service_close(__attribute__((unused)) int sig, siginfo_t *siginfo, __attribute__((unused))void *unused)
{
	// Only the signal from systemd or its' parent 
	pid_t ppid = getppid();
	if (siginfo->si_pid <= 1 || ppid == 1 || siginfo->si_pid == getppid())
		is_close = true;
}

static int start_socket_server()
{
	// bind signal
	struct sigaction act;
	act.sa_flags = SA_SIGINFO;
	act.sa_sigaction = wait_child;
	sigemptyset(&act.sa_mask);
	sigaction(SIGCHLD, &act, 0);
	act.sa_sigaction = service_close;
	sigemptyset(&act.sa_mask);
	sigaction(SIGTERM, &act, 0);

	// Set up the variables
	int server_sockfd, client_sockfd;
	socklen_t server_len, client_len;
	struct sockaddr_un server_address;
	struct sockaddr_un client_address;

	// Remove any old sockets and create an unnamed socket for the server
	unlink(socketfile);
	server_sockfd = socket(AF_UNIX, SOCK_STREAM | SOCK_CLOEXEC, 0);
	if (server_sockfd == -1) {
		syslog(LOG_ERR, "server socket create error. %m");
		return -1;
	} else {
		cout << "server socket create OK." << endl;
	}

	// Name the socket
	server_address.sun_family = AF_UNIX;
	strcpy(server_address.sun_path, socketfile);
	server_len = sizeof(server_address);
	int ret = bind(server_sockfd, (struct sockaddr *)&server_address, server_len);
	if (ret == -1) {
		syslog(LOG_ERR, "server socket bind error. %m");
		return -1;
	} else {
		cout << "server socket bind OK." << endl;
	}

	// Create a connection queue and wait for clients
	ret = listen(server_sockfd, 5);
	if (ret == -1) {
		syslog(LOG_ERR, "server socket listen error. %m");
		return -1;
	} else {
		cout << "server socket listen OK." << endl;
	}

	while(1) {
		// Accept a connection
		client_len = sizeof(client_address);
		client_sockfd = accept4(server_sockfd, (struct sockaddr *)&client_address, (socklen_t *)&client_len, SOCK_CLOEXEC);
		if (client_sockfd == -1) {
			if (errno != EINTR && errno != ERESTART) {
				syslog(LOG_ERR, "exec server accept client_sockfd error. %m");
			} else {
				if (is_close) {
					syslog(LOG_INFO, "exec server stop.");
					unlink(socketfile);
					close(server_sockfd);
					exit(0);
				} else {
					cout << "server accept interrupt signal, but not close" << endl;
				}
			}
			continue;
		}
		// receive command from client
		char buffer[1024] = {0};
		ssize_t read_len = read(client_sockfd, &buffer, 1024);
		if (read_len == -1) {
			syslog(LOG_ERR, "server read error. %m");
			close(client_sockfd);
			continue;
		} else {
			cout << "server read from client OK." << endl;
		}

		// create connection record
		ConnectionInfo connection;
		connection.client_sockfd = client_sockfd;
		if (-1 == pipe(connection.pipefd)) {
			syslog(LOG_ERR, "create pipe error. %m");
			close(client_sockfd);
			continue;
		}
		connection.cmd = string(buffer, read_len);

		// clone process to execute command
		char *stack_top = stack + STACK_SIZE;
		int pid = clone(deal_connection, stack_top, SIGCHLD | CLONE_NEWNET, &connection);
		if (pid == -1) {
			syslog(LOG_ERR, "clone failed. %m");
			continue;
		} else {
			cout << "server clone OK." << endl;
		}
		close(connection.pipefd[1]);    // close the write end of the pipe in the parent
		// add connection record to hash table
		connection_table[pid] = connection;
	}// end while(1)
}

int main()
{
	daemon(0, 1);
	start_socket_server();
	return 0;
}

```
