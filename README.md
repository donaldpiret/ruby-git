# Git Library for Ruby

Library for using Git in Ruby.

## Homepage

Git public hosting of the project source code is at:

http://github.com/schacon/ruby-git

## Install

You can install Ruby/Git like this:

    $ sudo gem install git
    
## Code Status

* [![Build Status](https://api.travis-ci.org/schacon/ruby-git.png)](https://travis-ci.org/schacon/ruby-git)
* [![Code Climate](https://codeclimate.com/github/schacon/ruby-git.png)](https://codeclimate.com/github/schacon/ruby-git)
* [![Gem Version](https://badge.fury.io/rb/git.png)](http://badge.fury.io/rb/git)
* [![Dependencies](https://gemnasium.com/schacon/ruby-git.png?travis)](https://gemnasium.com/schacon/ruby-git)

## Major Objects

**Git::Base** - The object returned from a `Git.open` or `Git.clone`. Most major actions are called from this object.

**Git::Object** - The base object for your tree, blob and commit objects, returned from `@git.gtree` or `@git.object` calls.  the `Git::AbstractObject` will have most of the calls in common for all those objects.

**Git::Diff** - returns from a `@git.diff` command.  It is an Enumerable that returns `Git::Diff:DiffFile` objects from which you can get per file patches and insertion/deletion statistics.  You can also get total statistics from the Git::Diff object directly.

**Git::Status** - returns from a `@git.status` command.  It is an Enumerable that returns
`Git:Status::StatusFile` objects for each object in git, which includes files in the working
directory, in the index and in the repository.  Similar to running 'git status' on the command line to determine untracked and changed files.

**Git::Branches** - Enumerable object that holds `Git::Branch objects`.  You can call .local or .remote on it to filter to just your local or remote branches.

**Git::Remote**- A reference to a remote repository that is tracked by this repository.

**Git::Log** - An Enumerable object that references all the `Git::Object::Commit` objects that encompass your log query, which can be constructed through methods on the `Git::Log object`,
like:

 `@git.log(20).object("some_file").since("2 weeks ago").between('v2.6', 'v2.7').each { |commit| [block] }`

## Examples

Here are a bunch of examples of how to use the Ruby/Git package.

Ruby < 1.9 will require rubygems to be loaded. 

```ruby
    require 'rubygems'
```

Require the 'git' gem.
```ruby
    require 'git'
```

Here are the operations that need read permission only.

```ruby
    g = Git.open(working_dir, :log => Logger.new(STDOUT))

    g.index
    g.index.readable?
    g.index.writable?
    g.repo
    g.dir

    g.log   # returns array of Git::Commit objects
    g.log.since('2 weeks ago')
    g.log.between('v2.5', 'v2.6')
    g.log.each {|l| puts l.sha }
    g.gblob('v2.5:Makefile').log.since('2 weeks ago')

    g.object('HEAD^').to_s  # git show / git rev-parse
    g.object('HEAD^').contents
    g.object('v2.5:Makefile').size
    g.object('v2.5:Makefile').sha

    g.gtree(treeish)
    g.gblob(treeish)
    g.gcommit(treeish)


    commit = g.gcommit('1cc8667014381')

    commit.gtree
    commit.parent.sha
    commit.parents.size
    commit.author.name
    commit.author.email
    commit.author.date.strftime("%m-%d-%y")
    commit.committer.name
    commit.date.strftime("%m-%d-%y")
    commit.message

    tree = g.gtree("HEAD^{tree}")

    tree.blobs
    tree.subtrees
    tree.children # blobs and subtrees

    g.revparse('v2.5:Makefile')

    g.branches # returns Git::Branch objects
    g.branches.local
    g.branches.remote
    g.branches[:master].gcommit
    g.branches['origin/master'].gcommit

    g.grep('hello')  # implies HEAD
    g.blob('v2.5:Makefile').grep('hello')
    g.tag('v2.5').grep('hello', 'docs/')

    g.diff(commit1, commit2).size
    g.diff(commit1, commit2).stats
    g.gtree('v2.5').diff('v2.6').insertions
    g.diff('gitsearch1', 'v2.5').path('lib/')
    g.diff('gitsearch1', @git.gtree('v2.5'))
    g.diff('gitsearch1', 'v2.5').path('docs/').patch
    g.gtree('v2.5').diff('v2.6').patch

    g.gtree('v2.5').diff('v2.6').each do |file_diff|
       puts file_diff.path
       puts file_diff.patch
       puts file_diff.blob(:src).contents
    end

    g.config('user.name')  # returns 'Scott Chacon'
    g.config # returns whole config hash

    g.tags # returns array of Git::Tag objects
```

And here are the operations that will need to write to your git repository.

```ruby
      g = Git.init
       Git.init('project')
       Git.init('/home/schacon/proj',
        { :git_dir => '/opt/git/proj.git',
           :index_file => '/tmp/index'} )

     g = Git.clone(URI, NAME, :path => '/tmp/checkout')
     g.config('user.name', 'Scott Chacon')
     g.config('user.email', 'email@email.com')

     g.add                                   # git add -- "."
     g.add(:all=>true)                       # git add --all -- "."
     g.add('file_path')                      # git add -- "file_path"
     g.add(['file_path_1', 'file_path_2'])   # git add -- "file_path_1" "file_path_2"


     g.remove('file.txt')
     g.remove(['file.txt', 'file2.txt'])

     g.commit('message')
     g.commit_all('message')

     g = Git.clone(repo, 'myrepo')
     g.chdir do
      new_file('test-file', 'blahblahblah')
      g.status.changed.each do |file|
       puts file.blob(:index).contents
      end
     end

     g.reset # defaults to HEAD
     g.reset_hard(Git::Commit)

     g.branch('new_branch') # creates new or fetches existing
     g.branch('new_branch').checkout
     g.branch('new_branch').delete
     g.branch('existing_branch').checkout

     g.checkout('new_branch')
     g.checkout(g.branch('new_branch'))

     g.branch(name).merge(branch2)
     g.branch(branch2).merge  # merges HEAD with branch2

     g.branch(name).in_branch(message) { # add files }  # auto-commits
     g.merge('new_branch')
     g.merge('origin/remote_branch')
     g.merge(g.branch('master'))
     g.merge([branch1, branch2])

     r = g.add_remote(name, uri)  # Git::Remote
     r = g.add_remote(name, Git::Base)  # Git::Remote

     g.remotes  # array of Git::Remotes
     g.remote(name).fetch
     g.remote(name).remove
     g.remote(name).merge
     g.remote(name).merge(branch)

     g.fetch
     g.fetch(g.remotes.first)

     g.pull
     g.pull(Git::Repo, Git::Branch) # fetch and a merge

     g.add_tag('tag_name') # returns Git::Tag

     g.repack

     g.push
     g.push(g.remote('name'))
```

Some examples of more low-level index and tree operations

```ruby
     g.with_temp_index do

       g.read_tree(tree3) # calls self.index.read_tree
       g.read_tree(tree1, :prefix => 'hi/')

       c = g.commit_tree('message')
       # or #
       t = g.write_tree
       c = g.commit_tree(t, :message => 'message', :parents => [sha1, sha2])

       g.branch('branch_name').update_ref(c)
       g.update_ref(branch, c)

       g.with_temp_working do # new blank working directory
         g.checkout
         g.checkout(another_index)
         g.commit # commits to temp_index
       end
     end

     g.set_index('/path/to/index')


     g.with_index(path) do
       # calls set_index, then switches back after
     end

     g.with_working(dir) do
     # calls set_working, then switches back after
     end

     g.with_temp_working(dir) do
       g.checkout_index(:prefix => dir, :path_limiter => path)
       # do file work
       g.commit # commits to index
     end
```

## License

licensed under MIT License Copyright (c) 2008  Scott Chacon. See LICENSE for further details.
