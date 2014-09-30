# Repository: Directory content tracking

As mentioned above, what Git does is quite rudimentary: it maintains snapshots of a directory’s contents. Much of its internal design can be understood in terms of this basic task.

The design of a Git repository in many ways mirrors the structure of a UNIX filesystem: A _filesystem_ begins with a root directory, which typically consists of other directories, most of which have leaf nodes, or _files_, that contain data. Meta-data about these files’ contents is stored both in the directory (the names), and in the i-nodes that reference the contents of those files (their size, type, permissions, etc). Each _i-node_ has a unique number that identifies the contents of its related file. And while you may have many directory entries pointing to a particular i-node (i.e., hard-links), it’s the i-node which “owns” the contents stored on your filesystem.

Internally, Git shares a strikingly similar structure, albeit with one or two key differences. First, it represents your file’s contents in _blobs_, which are also leaf nodes in something awfully close to a directory, called a _tree_. Just as an i-node is uniquely identified by a system-assigned number, a blob is named by computing the SHA1 hash id of its size and contents. For all intents and purposes this is just an arbitrary number, like an i-node, except that it has two additional properties: first, it verifies the blob’s contents will never change; and second, the same contents shall always be represented by the same blob, no matter where it appears: across commits, across repositories — even across the whole Internet. If multiple trees reference the same blob, this is just like hard-linking: the blob will not disappear from your repository as long as there is at least one link remaining to it.

The difference between a Git blob and a filesystem’s file is that a blob stores no metadata about its content. All such information is kept in the tree that holds the blob. One tree may know those contents as a file named “foo” that was created in August 2004, while another tree may know the same contents as a file named “bar” that was created five years later. In a normal filesystem, two files with the same contents but with such different metadata would always be represented as two independent files. Why this difference? Mainly, it’s because a filesystem is designed to support files that change, whereas Git is not. The fact that data is immutable in the Git repository is what makes all of this work and so a different design was needed. And as it turns out, this design allows for much more compact storage, since all objects having identical content can be shared, no matter where they are.