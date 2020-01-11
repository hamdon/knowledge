### svn相关操作-仓库同步到目录
增加仓库
```
svnadmin create /data/svn/web
```

同步到目录
```
/usr/bin/svn co svn://xxxxxxx/web /mnt/www/html/web/  --username hamdon --password xxxxxx --non-interactive
```

自动同步到网站根目录
```
#cd /data/svn/xyg/hooks
#vim post-commit
#!/bin/sh

# POST-COMMIT HOOK
#
# The post-commit hook is invoked after a commit.  Subversion runs
# this hook by invoking a program (script, executable, binary, etc.)
# named 'post-commit' (for which this file is a template) with the
# following ordered arguments:
#
#   [1] REPOS-PATH   (the path to this repository)
#   [2] REV          (the number of the revision just committed)
#   [3] TXN-NAME     (the name of the transaction that has become REV)
#
# Because the commit has already completed and cannot be undone,
# the exit code of the hook program is ignored.  The hook program
# can use the 'svnlook' utility to help it examine the
# newly-committed tree.
#
# The default working directory for the invocation is undefined, so
# the program should set one explicitly if it cares.
#
# On a Unix system, the normal procedure is to have 'post-commit'
# invoke other programs to do the real work, though it may do the
# work itself too.
#
# Note that 'post-commit' must be executable by the user(s) who will
# invoke it (typically the user httpd runs as), and that user must
# have filesystem-level permission to access the repository.
#
# On a Windows system, you should name the hook program
# 'post-commit.bat' or 'post-commit.exe',
# but the basic idea is the same.
#
# The hook program runs in an empty environment, unless the server is
# explicitly configured otherwise.  For example, a common problem is for
# the PATH environment variable to not be set to its usual value, so
# that subprograms fail to launch unless invoked via absolute path.
# If you're having unexpected problems with a hook program, the
# culprit may be unusual (or missing) environment variables.
#
# CAUTION:
# For security reasons, you MUST always properly quote arguments when
# you use them, as those arguments could contain whitespace or other
# problematic characters. Additionally, you should delimit the list
# of options with "--" before passing the arguments, so malicious
# clients cannot bootleg unexpected options to the commands your
# script aims to execute.
# For similar reasons, you should also add a trailing @ to URLs which
# are passed to SVN commands accepting URLs with peg revisions.
#
# Here is an example hook script, for a Unix /bin/sh interpreter.
# For more examples and pre-written hooks, see those in
# the Subversion repository at
# http://svn.apache.org/repos/asf/subversion/trunk/tools/hook-scripts/ and
# http://svn.apache.org/repos/asf/subversion/trunk/contrib/hook-scripts/


#REPOS="$1"
#REV="$2"
#TXN_NAME="$3"


export LANG="zh_CN.UTF-8"
REPOS="$1"
REV="$2"
#TXN_NAME="$3"

SVN=/usr/bin/svn
WEB=/mnt/www/html/web
LOG=/mnt/log/web-svn/auto_cool.log
SVNLOOK=/usr/bin/svnlook

AUTHOR=$($SVNLOOK author -r $REV "$REPOS")
CHANGEDDIRS=$($SVNLOOK dirs-changed $REPOS)
MESSAGE=$($SVNLOOK log -r $REV "$REPOS")

$SVN update $WEB --username hamdon --password xxxxxx --non-interactive
#......................
if [ $? == 0 ]
then
    echo  "AUTHOR:${AUTHOR},REV:${REV}" >> $LOG
    echo  "MESSAGE:$MESSAGE" >> $LOG
    echo  "CHANGEDDIRS ${CHANGEDDIRS}" >> $LOG
    echo `date` >> $LOG
    echo "##############################" >> $LOG
fi

```
