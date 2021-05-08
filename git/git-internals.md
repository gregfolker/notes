# Git Internals
## Overview
Git is fundamentally a **content-addressable filesystem** with a version control system user interface written on top of it

In the early days of `git`, the user interface was much more complex because it emphasized this filesystem rather than
attempting to be a polished VCS. Over the past few years however, the UI has been refined such that it's as clean and easy to use
as any other system out there today

## Plumbing and Porcelain
Because Git was initially a toolkit for a version control system rather than a full user-friendly VCS, it has a number of
subcommands that are intended for low-level work and were designed to be chained together UNIX-style or called from scripts.
These commands are generally referred to as Git's **plumbing** commands, while the more user-friendly commands (E.g., `checkout`,
`branch`, `remote` etc.) are referred to as Git's **porcelain** commands

This document focuses primarily on the plumbing commands within `git` and how they can be utilized to create new tools and custom
scripts that leverage the true power of the Git Version Control System

### `git init`
When you run `git init` in a new or existing directory, Git creates the `.git/` subdirectory, which is where almost everything that
`git` stores and manipulates is located. For example, if you want to backup or clone your repository, copying this single directory
with a command like `cp -r .git/ /path/to/my_backup/` gives you nearly everything you need

Here's what a newly-initialized `.git/` directory typically looks like:

```
$ ls -F1
config
description
HEAD
hooks/
info/
objects/
refs/
```

- `config`: The file that contains project-specific configuration options
- `info/`: The directory that keeps a global exclude file for ignored patterns you do not want to track in a `.gitignore` file
- `hooks/`: The directory that contains your client-side or server-side hook scripts
- `HEAD`: The file that is a pointer to the branch you currently have checked out
- `index`: The file where Git stores information about the staging area (Yet to be created)

## Git Objects
A content addressable filesystem is a filesystem that uses a simple key-value data store. This means you can insert any
kind of content into a Git repository and Git will give you back a unique key you can use to later access that same content

To illustrate this, we can use the plumbing command `git hash-object`, which takes some data, stores it inside `.git/objects/`,
(also known as the **object database**), and gives you back a unique key that refers to that data

```
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

The `-w` option tells the `hash-object` command to not just return the key but write that object to the database. The `--stdin`
option tells the command to get the content to be processed from `stdin`; otherwise, the command would expect a filename as an
argument. The output of this command is just a 40-character checksum hash (SHA1). If you now examine the `objects/` directory, you can
see that it now contains a file for that new content. This is how Git stores content initially - as a single file per piece of content
named with the SHA1 checksum of the content and its header

```
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

Using the command `git cat-file`, you can verify the contents of this Git object to be what we wrote to the database previously

```
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

The `-p` option instructs the `cat-file` command to first figure out the type of content before attempting to display it to the console

With this, you can add content to Git and pull it back out again. Doing this with content in files, you have the basic building blocks
for a version control system

```
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
```

Then, write some new contents to the file and run `hash-object` again to save it

```
$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
```

The object database now contains both versions of this new file. At this point, you can delete your local copy of `test.txt` and
actually use Git to retrieve it from the object database. Since we have two versions of it, we can get whichever version we want.
For example, to retrieve version 2:

```
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt
version 2
```

However, remembering the SHA1 key for each version of each file is not practical; plus, the filename itself isn't being stored with the
content. We could have easily renamed `test.txt` to `someothername.txt` since this type of object only stores the contents of the file
itself. This content type is called a `blob` in the content database. Using `git cat-file` with the `-t` option instructs Git to display
the object type of any object it has stored in the database

```
$ git cat-file -t 83baae61804e65cc73a7201a7252750c76066a30
blob
```

## Tree Objects
The `tree` object allows you to store a group of files together. Git stores files in a similar manner to a UNIX filesystem. All of
the content is stored as a tree where a leaf is just a `blob` of data. The most recent tree in a project may look something like
the following:

```
$ git cat-file -p main^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib
```

The `main^{tree}` syntax is used to specify the object that the last commit points to on the `main` branch. Note how we can tell that
`lib` is a directory and not a file because it has the type `tree` instead of `blob`. You can also use `git cat-file` to observe that
`lib` is actually a pointer to another tree:

```
$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb
```

Git creates a tree by taking the state of your staging area or index and creating a series of tree objects based on it. So, in order
to create a tree object, you first have to set up an index by staging some files. This can be accomplished with the plumbing command
`update-index`

```
$ git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt
```

Where `100644` is the new filemode, `83baae61804e65cc73a7201a7252750c76066a30` is the SHA1 hash of the file, and `test.txt` is the
filename itself. The `--add` option is required because the file does not exist yet in the staging area and the `--cacheinfo`
option is required because `test.txt` isn't present in the current directory but instead in the object database

From here, the command `git write-tree` can be used to write the staging area out as a tree object

```
$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
100644 blob 83baae61804e65cc73a7201a7252750c76066a30      test.txt
```


`git update-index --add` can be used to add files to the newly created index as well:

```
$ echo 'new file' > new.txt
$ git update-index --add new.txt
```

`git update-index` can also be used to create new versions of existing files, for example:

```
$ git update-index --add --cacheinfo 100644 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
```

Now, `git write-tree` can be used to write the staging area out to a new tree object with the new file `new.txt` and the
new version of `test.txt`

```
$ git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
```

Reading trees into the staging area can be done as well using the `read-tree` command. If we wanted to take the first tree we created,
which had a hash of `d8329fc1cc938780ffdd9f94e0d364e0ea74f579`, and add it as a subdirectory to the project as `bak`, it could be done
with the following combination of commands:

```
$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614
$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579      bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
```

## Commit Objects
At this point, we now have three separate trees that represent different snapshots of the project we want to track. These snapshots
can be accessed using their respective hashes at any point, but this isn't very practical from an end-user standpoint. Not to mention
we still do not have any information about who/what/when/why these snapshots were created. This is what **commit objects** are for

To create a commit object, use the `commit-tree` command and specify the SHA1 hash of which tree you want to create a commit of. For
example, to create a commit object of the first tree we created above:

```
$ echo 'First commit' | git commit-tree d8329f
fdf4fc3344e67ab068f836878b6c4951e3b15f3d
```

This new commit object can now be observed using `git cat-file`:

```
$ git cat-file -p fdf4fc3
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Greg Folker <gregfolker.dev@gmail.com> 1243040974 -0700
committer Greg Folker <gregfolker.dev@gmail.com> 1243040974 -0700

First commit
```

The format of a commit object is actually very simple; it specifies the top level tree for the snapshot that was created, the parent
commits (if any), the author and committer information, a single blank line, and finally the commit message itself

We can create commit objects for the other trees we have in our project as well:

```
$ echo 'Second commit' | git commit-tree 0155eb -p fdf4fc3
cac0cab538b970a37ea1e769cbbde608743bc96d
$ echo 'Third commit'  | git commit-tree 3c4e9c -p cac0cab
1a410efbd13591db07496601ebc7a059dd55cfe9
```

Each of these three commits points to one of the three snapshots that was created using `git write-tree` above. This is where `git log`
gets its information to display

```
$ git log --stat 1a410e
commit 1a410efbd13591db07496601ebc7a059dd55cfe9
Author: Greg Folker <schacon@gmail.com>
Date:   Fri May 22 18:15:24 2009 -0700

	Third commit

 bak/test.txt | 1 +
 1 file changed, 1 insertion(+)

commit cac0cab538b970a37ea1e769cbbde608743bc96d
Author: Greg Folker <schacon@gmail.com>
Date:   Fri May 22 18:14:29 2009 -0700

	Second commit

 new.txt  | 1 +
 test.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)

commit fdf4fc3344e67ab068f836878b6c4951e3b15f3d
Author: Greg Folker <schacon@gmail.com>
Date:   Fri May 22 18:09:34 2009 -0700

    First commit

 test.txt | 1 +
 1 file changed, 1 insertion(+)
```

## Object Storage
There is a header stored with every object you commit to your Git object database. Let's see how a blob object is stored by
interactively using the Ruby scripting language

Ruby mode can be started using the `irb` command:

```
$ irb
>> content = "what is up, doc?"
=> "what is up, doc?"
```

Git first constructs a header which starts by identifying the type of the object. Git adds a space followed by the size (in bytes) of
the content and by appending a terminating null byte:

```
>> header = "blob #{content.bytesize}\0"
=> "blob 16\u0000"
```

Git then concatenates the header with the original content and calculates a new SHA1 checksum of the new content

```
>> store = header + content
=> "blob 16\u0000what is up, doc?"
>> require 'digest/sha1'
=> true
>> sha1 = Digest::SHA1.hexdigest(store)
=> "bd9dbf5aae1a3862dd1526723246b20206e5fc37"
```

We can use the output of `git hash-object` to compare against

```
$ echo -n "what is up, doc?" | git hash-object --stdin
bd9dbf5aae1a3862dd1526723246b20206e5fc37
```

Note the `-n` option on the `echo` command is needed to prevent adding a newline character to the input stream
