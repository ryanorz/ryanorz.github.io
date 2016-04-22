---
layout:     post
title:      "Sphinx: full text seach engine"
date:       2016-04-19
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - Database
    - linux
--- 

# install

install mariadb server first. Then install the engine.
Fedora 23:    `# dnf install sphinx`
Ubuntu 16.04: `# apt-get install sphinxsearch`

# Config and import data

config file:
Fedora 23:    `/etc/sphinx/sphinx.conf`
Ubuntu 16.04: `/etc/sphinxsearch/sphinx.conf`

add test data to database:

```sql
DROP TABLE IF EXISTS sphinx.documents;
CREATE TABLE sphinx.documents
(
        id              INTEGER PRIMARY KEY NOT NULL AUTO_INCREMENT,
        group_id        INTEGER NOT NULL,
        group_id2       INTEGER NOT NULL,
        date_added      DATETIME NOT NULL,
        title           VARCHAR(255) NOT NULL,
        content         TEXT NOT NULL
);

REPLACE INTO sphinx.documents ( id, group_id, group_id2, date_added, title, content ) VALUES
        ( 1, 1, 5, NOW(), 'test one', 'this is my test document number one. also checking search within phrases.' ),
        ( 2, 1, 6, NOW(), 'test two', 'this is my test document number two' ),
        ( 3, 2, 7, NOW(), 'another doc', 'this is another group' ),
        ( 4, 2, 8, NOW(), 'doc number four', 'this is to test groups' );

DROP TABLE IF EXISTS test.tags;
CREATE TABLE test.tags
(
        docid INTEGER NOT NULL,
        tagid INTEGER NOT NULL,
        UNIQUE(docid,tagid)
);

INSERT INTO test.tags VALUES
        (1,1), (1,3), (1,5), (1,7),
        (2,6), (2,4), (2,2),
        (3,15),
        (4,7), (4,40);
```

# generate index and query index

```sh
indexer --all
systemctl start searchd
```
query index

```sql
$ mysql -h0 -P9306

SELECT * FROM test1 WHERE MATCH('my document');

INSERT INTO testrt VALUES (1, 'this is', 'a sample text', 11);

INSERT INTO testrt VALUES (2, 'some more', 'text here', 22);

SELECT gid/11 FROM testrt WHERE MATCH('text') GROUP BY gid;

SELECT * FROM testrt ORDER BY gid DESC;

SHOW TABLES;

SELECT *, WEIGHT() FROM test1 WHERE MATCH('"document one"/1');SHOW META;

SET profiling=1;SELECT * FROM test1 WHERE id IN (1,2,4);SHOW PROFILE;

SELECT id, id%3 idd FROM test1 WHERE MATCH('this is | nothing') GROUP BY idd;SHOW PROFILE;

SELECT id FROM test1 WHERE MATCH('is this a good plan?');SHOW PLAN;

SELECT COUNT(*) c, id%3 idd FROM test1 GROUP BY idd HAVING COUNT(*)>1;

SELECT COUNT(*) FROM test1;

CALL KEYWORDS ('one two three', 'test1');

CALL KEYWORDS ('one two three', 'test1', 1);
```
