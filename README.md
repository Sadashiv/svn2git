Convert svn to git repo with history
====================================

Referance link: https://john.albin.net/git/convert-subversion-to-git

1. svn co [svn repo url] svnrepo
    cd svnrepo
    svn log -q | awk -F '|' '/^r/ {sub("^ ", "", $2); sub(" $", "", $2); print $2" = "$2" <"$2">"}' | sort -u > authors-transform.txt

    change:

    jwilkins = jwilkins <jwilkins>
    into this:

    jwilkins = John Albin Wilkins <johnalbin@example.com>

2. git svn clone  [svn repo url] --no-metadata -A ~/steering/authors-transform.txt --stdlayout ~/svntemp

3. Convert svn:ignore properties to .gitignore
    If your svn repo was using svn:ignore properties, you can easily convert this to a .gitignore file using:

    cd ~/svntemp
    git svn show-ignore > .gitignore
    git add .gitignore
    git commit -m 'Convert svn:ignore properties to .gitignore.'

4. Push repository to a bare git repository
    First, create a bare repository and make its default branch match svn’s “trunk” branch name.

    git init --bare ~/new-bare.git
    cd ~/new-bare.git
    git symbolic-ref HEAD refs/heads/trunk
    Then push the temp repository to the new bare repository.

    cd ~/temp
    git remote add bare ~/new-bare.git
    git config remote.bare.push 'refs/remotes/*:refs/heads/*'
    git push bare
    You can now safely delete the ~/temp repository.

5. Rename “trunk” branch to “master”
    Your main development branch will be named “trunk” which matches the name it was in Subversion. You’ll want to rename it to Git’s standard “master” branch using:

    cd ~/new-bare.git
    git branch -m trunk master

6. Clean up branches and tags
    git-svn makes all of Subversions tags into very-short branches in Git of the form “tags/name”. You’ll want to convert all those branches into actual Git tags using:

    cd ~/new-bare.git
    git for-each-ref --format='%(refname)' refs/heads/tags |
    cut -d / -f 4 |
    while read ref
    do
      git tag "$ref" "refs/heads/tags/$ref";
      git branch -D "tags/$ref";
    done
    This step will take a bit of typing. :-) But, don’t worry; your unix shell will provide a > secondary prompt for the extra-long command that starts with git for-each-ref.





