A prepopulated mysql container ?
================================

Yeah why not, i like my test faster :D

But how ?
---------

The mysql/mariadb container image contain an init script that will execute everything in `/docker-entrypoint-initdb.d/`

see `Initializing a fresh instance` @ https://hub.docker.com/_/mariadb/

So let's run this initialization in a multi-stage build and copy the initialized DB in the new image :D

(see comments in dockerfile for details of the hack) 

Try it!
======

Clone this, then...

```
}> docker build --tag my-prepopulated-image .
...

}> docker run -d --rm --name my-container my-prepopulated-image
...

}> docker logs my-container

(there was is initialization here, therefore we win)

2018-06-08 21:15:55 0 [Note] mysqld (mysqld 10.3.7-MariaDB-1:10.3.7+maria~jessie) starting as process 1 ...
2018-06-08 21:15:55 0 [Note] InnoDB: Using Linux native AIO
2018-06-08 21:15:55 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2018-06-08 21:15:55 0 [Note] InnoDB: Uses event mutexes
2018-06-08 21:15:55 0 [Note] InnoDB: Compressed tables use zlib 1.2.8
2018-06-08 21:15:55 0 [Note] InnoDB: Number of pools: 1
2018-06-08 21:15:55 0 [Note] InnoDB: Using SSE2 crc32 instructions
2018-06-08 21:15:55 0 [Note] InnoDB: Initializing buffer pool, total size = 256M, instances = 1, chunk size = 128M
2018-06-08 21:15:55 0 [Note] InnoDB: Completed initialization of buffer pool
2018-06-08 21:15:55 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2018-06-08 21:15:56 0 [Note] InnoDB: 128 out of 128 rollback segments are active.
2018-06-08 21:15:56 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2018-06-08 21:15:56 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2018-06-08 21:15:56 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2018-06-08 21:15:56 0 [Note] InnoDB: 10.3.7 started; log sequence number 1639605; transaction id 39
2018-06-08 21:15:56 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2018-06-08 21:15:56 0 [Note] Plugin 'FEEDBACK' is disabled.
2018-06-08 21:15:56 0 [Note] Server socket created on IP: '::'.
2018-06-08 21:15:56 0 [Warning] 'proxies_priv' entry '@% root@9977cc179be0' ignored in --skip-name-resolve mode.
2018-06-08 21:15:56 0 [Note] InnoDB: Buffer pool(s) load completed at 180608 21:15:56
2018-06-08 21:15:56 0 [Note] Reading of all Master_info entries succeded
2018-06-08 21:15:56 0 [Note] Added new Master_info '' to hash table
2018-06-08 21:15:56 0 [Note] mysqld: ready for connections.
Version: '10.3.7-MariaDB-1:10.3.7+maria~jessie'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution

}> docker run -it --rm --link my-container mariadb:latest mysql -hmy-container -uroot -proot myexample -e "select * from mytable;"
+---------+
| myfield |
+---------+
| Hello   |
| Dolly   |
+---------+
```
