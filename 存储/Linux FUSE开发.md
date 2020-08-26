# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [github libfuse](https://github.com/libfuse/libfuse)
> [FUSE简介（译）](https://blog.csdn.net/coroutines/article/details/39497601)
> [用户态文件系统fuse学习](https://blog.csdn.net/ty_laurel/article/details/51685193)
> [使用 FUSE 开发自己的文件系统](https://www.ibm.com/developerworks/cn/linux/l-fuse/)
> [fuse接口用法说明](https://blog.csdn.net/t3swing/article/details/82901628)
> [linux的readdir和readdir_r函数](https://blog.csdn.net/qq_40732350/article/details/81986548)
> [基于fuse文件系统优化方法总结[附带详细说明]](https://blog.csdn.net/ifyourmm/article/details/51657335)

# 介绍
FUSE指的是用户态文件系统，FUSE在内核中存在一个驱动，用户态对文件系统的操作，通过FUSE驱动转发到用户态程序，这样就可以在用户态实现文件系统操作，传统的Linux文件系统存在于内核中，比如Linux下对NTFS提供读写支持的NTFS-3G软件包就是利用FUSE实现的，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190502213312135.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 实现
实现FUSE也就是实现FUSE定义的一套函数集，通过这组函数虚拟出一个文件系统，不管这个文件系统存在什么地方，以一种什么样的形式，
```c
struct fuse_operations {
    int (*getattr) (const char *, struct stat *);
    int (*readlink) (const char *, char *, size_t);
    int (*getdir) (const char *, fuse_dirh_t, fuse_dirfil_t);
    int (*mknod) (const char *, mode_t, dev_t);
    int (*mkdir) (const char *, mode_t);
    int (*unlink) (const char *);
    int (*rmdir) (const char *);
    int (*symlink) (const char *, const char *);
    int (*rename) (const char *, const char *);
    int (*link) (const char *, const char *);
    int (*chmod) (const char *, mode_t);
    int (*chown) (const char *, uid_t, gid_t);
    int (*truncate) (const char *, off_t);
    int (*utime) (const char *, struct utimbuf *);
    int (*open) (const char *, struct fuse_file_info *);
    int (*read) (const char *, char *, size_t, off_t, struct fuse_file_info *);
    int (*write) (const char *, const char *, size_t, off_t,struct fuse_file_info *);
    int (*statfs) (const char *, struct statfs *);
    int (*flush) (const char *, struct fuse_file_info *);
    int (*release) (const char *, struct fuse_file_info *);
    int (*fsync) (const char *, int, struct fuse_file_info *);
    int (*setxattr) (const char *, const char *, const char *, size_t, int);
    int (*getxattr) (const char *, const char *, char *, size_t);
    int (*listxattr) (const char *, char *, size_t);
    int (*removexattr) (const char *, const char *);
};
```
1. getattr: int (*getattr) (const char *, struct stat *);
这个函数与 stat() 类似。st_dev 和 st_blksize 域都可以忽略。st_ino 域也会被忽略，除非在执行 mount 时指定了 use_ino 选项。
2. readlink: int (*readlink) (const char *, char *, size_t);
这个函数会读取一个符号链接的目标。缓冲区应该是一个以 null 结束的字符串。缓冲区的大小参数包括这个 null 结束字符的空间。如果链接名太长，不能保存到缓冲区中，就应该被截断。成功时的返回值应该是 “0”。
3. getdir: int (*getdir) (const char *, fuse_dirh_t, fuse_dirfil_t);
这个函数会读取一个目录中的内容。这个操作实际上是在一次调用中执行 opendir()、readdir()、...、closedir() 序列。对于每个目录项来说，都应该调用 filldir() 函数。
4. mknod: int (*mknod) (const char *, mode_t, dev_t);
这个函数会创建一个文件节点。此处没有 create() 操作；mknod() 会在创建非目录、非符号链接的节点时调用。
5. mkdir: int (*mkdir) (const char *, mode_t);
创建一个目录。
6. rmdir: int (*rmdir) (const char *);
删除一个目录。
7. unlink: int (*unlink) (const char *);
删除一个文件。
8. rename: int (*rename) (const char *, const char *);
重命名一个文件。
9. symlink: int (*symlink) (const char *, const char *);
这个函数用来创建一个符号链接。
10. link: int (*link) (const char *, const char *);
这个函数创建一个到文件的硬链接。
11. chmod: int (*chmod) (const char *, mode_t);
12. chown: int (*chown) (const char *, uid_t, gid_t);
13. truncate: int (*truncate) (const char *, off_t);
14. utime: int (*utime) (const char *, struct utimbuf *);
这 4 个函数分别用来修改文件的权限位、属主和用户、大小以及文件的访问/修改时间。
15. open: int (*open) (const char *, struct fuse_file_info *);
这是文件的打开操作。对 open() 函数不能传递创建或截断标记（O_CREAT、O_EXCL、O_TRUNC）。这个函数应该检查是否允许执行给定的标记的操作。另外，open() 也可能在 fuse_file_info 结构中返回任意的文件句柄，这会传递给所有的文件操作。
16. read: int (*read) (const char *, char *, size_t, off_t, struct fuse_file_info *);
这个函数从一个打开文件中读取数据。除非碰到 EOF 或出现错误，否则 read() 应该返回所请求的字节数的数据；否则，其余数据都会被替换成 0。一个例外是在执行 mount 命令时指定了 direct_io 选项，在这种情况中 read() 系统调用的返回值会影响这个操作的返回值。
17. write: int (*write) (const char *, const char *, size_t, off_t, struct fuse_file_info *);
这个函数将数据写入一个打开的文件中。除非碰到 EOF 或出现错误，否则 write() 应该返回所请求的字节数的数据。一个例外是在执行 mount 命令时指定了 direct_io 选项（这于 read() 操作的情况类似）。
18. statfs: int (*statfs) (const char *, struct statfs *);
这个函数获取文件系统的统计信息。f_type 和 f_fsid 域都会被忽略。
19. flush: int (*flush) (const char *, struct fuse_file_info *);
这表示要刷新缓存数据。它并不等于 fsync() 函数 —— 也不是请求同步脏数据。每次对一个文件描述符执行 close() 函数时，都会调用 flush()；因此如果文件系统希望在 close() 中返回写错误，并且这个文件已经缓存了脏数据，那么此处就是回写数据并返回错误的好地方。由于很多应用程序都会忽略 close() 错误，因此这通常用处不大。
注意：我们也可以对一个 open() 多次调用 flush() 方法。如果由于调用了 dup()、dup2() 或 fork() 而产生多个文件描述符指向一个打开文件的情况，就可能会需要这种用法。我们无法确定哪个 flush 操作是最后一次操作，因此每个 flush 都应该同等地对待。多个写刷新序列相当罕见，因此这并不是什么问题。
20. release: int (*release) (const char *, struct fuse_file_info *);
这个函数释放一个打开文件。release() 是在对一个打开文件没有其他引用时调用的 —— 此时所有的文件描述符都会被关闭，所有的内存映射都会被取消。对于每个 open() 调用来说，都必须有一个使用完全相同标记和文件描述符的 release() 调用。对一个文件打开多次是可能的，在这种情况中只会考虑最后一次 release，然后就不能再对这个文件执行更多的读/写操作了。release 的返回值会被忽略。
21. fsync: int (*fsync) (const char *, int, struct fuse_file_info *);
这个函数用来同步文件内容。如果 datasync 参数为非 0，那么就只会刷新用户数据，而不会刷新元数据。
22. setxattr: int (*setxattr) (const char *, const char *, const char *, size_t, int);
23. getxattr: int (*getxattr) (const char *, const char *, char *, size_t);
24. listxattr: int (*listxattr) (const char *, char *, size_t);
25. removexattr: int (*removexattr) (const char *, const char *);
这些函数分别用来设置、获取、列出和删除扩展属性。
# 性能优化
根据我们自研的文件系统特性，采取下面三个措施，提高顺序读写性能，
- 延长元数据有效时间，–o entry_timeout=T –o attr_timeout=T
- 扩大每次写入页面数，–o big_write –o max_write=N
- 使用DirectIO取代BufferIO，-o direct_io
# 示例
libfuse自带的hello，构造了一个只有一个可读文件的文件系统，文件名和文件内容来自于main的输入参数，
```c
/*
  FUSE: Filesystem in Userspace
  Copyright (C) 2001-2007  Miklos Szeredi <miklos@szeredi.hu>
  This program can be distributed under the terms of the GNU GPL.
  See the file COPYING.
*/

/** @file
 *
 * minimal example filesystem using high-level API
 *
 * Compile with:
 *
 *     gcc -Wall hello.c `pkg-config fuse3 --cflags --libs` -o hello
 *
 * ## Source code ##
 * \include hello.c
 */


#define FUSE_USE_VERSION 31

#include <fuse.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <fcntl.h>
#include <stddef.h>
#include <assert.h>

/*
 * Command line options
 *
 * We can't set default values for the char* fields here because
 * fuse_opt_parse would attempt to free() them when the user specifies
 * different values on the command line.
 */
static struct options {
	const char *filename;
	const char *contents;
	int show_help;
} options;

#define OPTION(t, p)                           \
    { t, offsetof(struct options, p), 1 }
static const struct fuse_opt option_spec[] = {
	OPTION("--name=%s", filename),
	OPTION("--contents=%s", contents),
	OPTION("-h", show_help),
	OPTION("--help", show_help),
	FUSE_OPT_END
};

static void *hello_init(struct fuse_conn_info *conn,
			struct fuse_config *cfg)
{
	(void) conn;
	cfg->kernel_cache = 1;
	return NULL;
}

static int hello_getattr(const char *path, struct stat *stbuf,
			 struct fuse_file_info *fi)
{
	(void) fi;
	int res = 0;

	memset(stbuf, 0, sizeof(struct stat));
	if (strcmp(path, "/") == 0) {
		stbuf->st_mode = S_IFDIR | 0755;
		stbuf->st_nlink = 2;
	} else if (strcmp(path+1, options.filename) == 0) {
		stbuf->st_mode = S_IFREG | 0444;
		stbuf->st_nlink = 1;
		stbuf->st_size = strlen(options.contents);
	} else
		res = -ENOENT;

	return res;
}

static int hello_readdir(const char *path, void *buf, fuse_fill_dir_t filler,
			 off_t offset, struct fuse_file_info *fi,
			 enum fuse_readdir_flags flags)
{
	(void) offset;
	(void) fi;
	(void) flags;

	if (strcmp(path, "/") != 0)
		return -ENOENT;

	filler(buf, ".", NULL, 0, 0);
	filler(buf, "..", NULL, 0, 0);
	filler(buf, options.filename, NULL, 0, 0);

	return 0;
}

static int hello_open(const char *path, struct fuse_file_info *fi)
{
	if (strcmp(path+1, options.filename) != 0)
		return -ENOENT;

	if ((fi->flags & O_ACCMODE) != O_RDONLY)
		return -EACCES;

	return 0;
}

static int hello_read(const char *path, char *buf, size_t size, off_t offset,
		      struct fuse_file_info *fi)
{
	size_t len;
	(void) fi;
	if(strcmp(path+1, options.filename) != 0)
		return -ENOENT;

	len = strlen(options.contents);
	if (offset < len) {
		if (offset + size > len)
			size = len - offset;
		memcpy(buf, options.contents + offset, size);
	} else
		size = 0;

	return size;
}

static struct fuse_operations hello_oper = {
	.init           = hello_init,
	.getattr	= hello_getattr,
	.readdir	= hello_readdir,
	.open		= hello_open,
	.read		= hello_read,
};

static void show_help(const char *progname)
{
	printf("usage: %s [options] <mountpoint>\n\n", progname);
	printf("File-system specific options:\n"
	       "    --name=<s>          Name of the \"hello\" file\n"
	       "                        (default: \"hello\")\n"
	       "    --contents=<s>      Contents \"hello\" file\n"
	       "                        (default \"Hello, World!\\n\")\n"
	       "\n");
}

int main(int argc, char *argv[])
{
	int ret;
	struct fuse_args args = FUSE_ARGS_INIT(argc, argv);

	/* Set defaults -- we have to use strdup so that
	   fuse_opt_parse can free the defaults if other
	   values are specified */
	options.filename = strdup("hello");
	options.contents = strdup("Hello World!\n");

	/* Parse options */
	if (fuse_opt_parse(&args, &options, option_spec, NULL) == -1)
		return 1;

	/* When --help is specified, first print our own file-system
	   specific help text, then signal fuse_main to show
	   additional help (by adding `--help` to the options again)
	   without usage: line (by setting argv[0] to the empty
	   string) */
	if (options.show_help) {
		show_help(argv[0]);
		assert(fuse_opt_add_arg(&args, "--help") == 0);
		args.argv[0][0] = '\0';
	}

	ret = fuse_main(args.argc, args.argv, &hello_oper, NULL);
	fuse_opt_free_args(&args);
	return ret;
}
```
参考`fusexmp.c`，全面的实现，
```c
/*
  FUSE: Filesystem in Userspace
  Copyright (C) 2001-2007  Miklos Szeredi <miklos@szeredi.hu>
  Copyright (C) 2011       Sebastian Pipping <sebastian@pipping.org>

  This program can be distributed under the terms of the GNU GPL.
  See the file COPYING.

  gcc -Wall fusexmp.c `pkg-config fuse --cflags --libs` -o fusexmp
*/

#define FUSE_USE_VERSION 26

#ifdef HAVE_CONFIG_H
#include <config.h>
#endif

#ifdef linux
/* For pread()/pwrite()/utimensat() */
#define _XOPEN_SOURCE 700
#endif

#include <fuse.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <dirent.h>
#include <errno.h>
#include <sys/time.h>
#ifdef HAVE_SETXATTR
#include <sys/xattr.h>
#endif

static int xmp_getattr(const char *path, struct stat *stbuf)
{
	int res;

	res = lstat(path, stbuf);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_access(const char *path, int mask)
{
	int res;

	res = access(path, mask);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_readlink(const char *path, char *buf, size_t size)
{
	int res;

	res = readlink(path, buf, size - 1);
	if (res == -1)
		return -errno;

	buf[res] = '\0';
	return 0;
}


static int xmp_readdir(const char *path, void *buf, fuse_fill_dir_t filler,
		       off_t offset, struct fuse_file_info *fi)
{
	DIR *dp;
	struct dirent *de;

	(void) offset;
	(void) fi;

	dp = opendir(path);
	if (dp == NULL)
		return -errno;

	while ((de = readdir(dp)) != NULL) {
		struct stat st;
		memset(&st, 0, sizeof(st));
		st.st_ino = de->d_ino;
		st.st_mode = de->d_type << 12;
		if (filler(buf, de->d_name, &st, 0))
			break;
	}

	closedir(dp);
	return 0;
}

static int xmp_mknod(const char *path, mode_t mode, dev_t rdev)
{
	int res;

	/* On Linux this could just be 'mknod(path, mode, rdev)' but this
	   is more portable */
	if (S_ISREG(mode)) {
		res = open(path, O_CREAT | O_EXCL | O_WRONLY, mode);
		if (res >= 0)
			res = close(res);
	} else if (S_ISFIFO(mode))
		res = mkfifo(path, mode);
	else
		res = mknod(path, mode, rdev);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_mkdir(const char *path, mode_t mode)
{
	int res;

	res = mkdir(path, mode);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_unlink(const char *path)
{
	int res;

	res = unlink(path);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_rmdir(const char *path)
{
	int res;

	res = rmdir(path);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_symlink(const char *from, const char *to)
{
	int res;

	res = symlink(from, to);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_rename(const char *from, const char *to)
{
	int res;

	res = rename(from, to);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_link(const char *from, const char *to)
{
	int res;

	res = link(from, to);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_chmod(const char *path, mode_t mode)
{
	int res;

	res = chmod(path, mode);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_chown(const char *path, uid_t uid, gid_t gid)
{
	int res;

	res = lchown(path, uid, gid);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_truncate(const char *path, off_t size)
{
	int res;

	res = truncate(path, size);
	if (res == -1)
		return -errno;

	return 0;
}

#ifdef HAVE_UTIMENSAT
static int xmp_utimens(const char *path, const struct timespec ts[2])
{
	int res;

	/* don't use utime/utimes since they follow symlinks */
	res = utimensat(0, path, ts, AT_SYMLINK_NOFOLLOW);
	if (res == -1)
		return -errno;

	return 0;
}
#endif

static int xmp_open(const char *path, struct fuse_file_info *fi)
{
	int res;

	res = open(path, fi->flags);
	if (res == -1)
		return -errno;

	close(res);
	return 0;
}

static int xmp_read(const char *path, char *buf, size_t size, off_t offset,
		    struct fuse_file_info *fi)
{
	int fd;
	int res;

	(void) fi;
	fd = open(path, O_RDONLY);
	if (fd == -1)
		return -errno;

	res = pread(fd, buf, size, offset);
	if (res == -1)
		res = -errno;

	close(fd);
	return res;
}

static int xmp_write(const char *path, const char *buf, size_t size,
		     off_t offset, struct fuse_file_info *fi)
{
	int fd;
	int res;

	(void) fi;
	fd = open(path, O_WRONLY);
	if (fd == -1)
		return -errno;

	res = pwrite(fd, buf, size, offset);
	if (res == -1)
		res = -errno;

	close(fd);
	return res;
}

static int xmp_statfs(const char *path, struct statvfs *stbuf)
{
	int res;

	res = statvfs(path, stbuf);
	if (res == -1)
		return -errno;

	return 0;
}

static int xmp_release(const char *path, struct fuse_file_info *fi)
{
	/* Just a stub.	 This method is optional and can safely be left
	   unimplemented */

	(void) path;
	(void) fi;
	return 0;
}

static int xmp_fsync(const char *path, int isdatasync,
		     struct fuse_file_info *fi)
{
	/* Just a stub.	 This method is optional and can safely be left
	   unimplemented */

	(void) path;
	(void) isdatasync;
	(void) fi;
	return 0;
}

#ifdef HAVE_POSIX_FALLOCATE
static int xmp_fallocate(const char *path, int mode,
			off_t offset, off_t length, struct fuse_file_info *fi)
{
	int fd;
	int res;

	(void) fi;

	if (mode)
		return -EOPNOTSUPP;

	fd = open(path, O_WRONLY);
	if (fd == -1)
		return -errno;

	res = -posix_fallocate(fd, offset, length);

	close(fd);
	return res;
}
#endif

#ifdef HAVE_SETXATTR
/* xattr operations are optional and can safely be left unimplemented */
static int xmp_setxattr(const char *path, const char *name, const char *value,
			size_t size, int flags)
{
	int res = lsetxattr(path, name, value, size, flags);
	if (res == -1)
		return -errno;
	return 0;
}

static int xmp_getxattr(const char *path, const char *name, char *value,
			size_t size)
{
	int res = lgetxattr(path, name, value, size);
	if (res == -1)
		return -errno;
	return res;
}

static int xmp_listxattr(const char *path, char *list, size_t size)
{
	int res = llistxattr(path, list, size);
	if (res == -1)
		return -errno;
	return res;
}

static int xmp_removexattr(const char *path, const char *name)
{
	int res = lremovexattr(path, name);
	if (res == -1)
		return -errno;
	return 0;
}
#endif /* HAVE_SETXATTR */

static struct fuse_operations xmp_oper = {
	.getattr	= xmp_getattr,
	.access		= xmp_access,
	.readlink	= xmp_readlink,
	.readdir	= xmp_readdir,
	.mknod		= xmp_mknod,
	.mkdir		= xmp_mkdir,
	.symlink	= xmp_symlink,
	.unlink		= xmp_unlink,
	.rmdir		= xmp_rmdir,
	.rename		= xmp_rename,
	.link		= xmp_link,
	.chmod		= xmp_chmod,
	.chown		= xmp_chown,
	.truncate	= xmp_truncate,
#ifdef HAVE_UTIMENSAT
	.utimens	= xmp_utimens,
#endif
	.open		= xmp_open,
	.read		= xmp_read,
	.write		= xmp_write,
	.statfs		= xmp_statfs,
	.release	= xmp_release,
	.fsync		= xmp_fsync,
#ifdef HAVE_POSIX_FALLOCATE
	.fallocate	= xmp_fallocate,
#endif
#ifdef HAVE_SETXATTR
	.setxattr	= xmp_setxattr,
	.getxattr	= xmp_getxattr,
	.listxattr	= xmp_listxattr,
	.removexattr	= xmp_removexattr,
#endif
};

int main(int argc, char *argv[])
{
	umask(0);
	return fuse_main(argc, argv, &xmp_oper, NULL);
}

```

