<h1><a name="wasi_filesystem">World wasi-filesystem</a></h1>
<ul>
<li>Imports:
<ul>
<li>interface <a href="#wasi_poll"><code>wasi-poll</code></a></li>
<li>interface <a href="#wasi_io"><code>wasi-io</code></a></li>
<li>interface <a href="#wasi_wall_clock"><code>wasi-wall-clock</code></a></li>
<li>interface <a href="#wasi_filesystem"><code>wasi-filesystem</code></a></li>
</ul>
</li>
</ul>
<h2><a name="wasi_poll">Import interface wasi-poll</a></h2>
<p>WASI Poll is a poll API intended to let users wait for I/O events on
multiple handles at once.</p>
<hr />
<h3>Types</h3>
<h4><a name="pollable"><code>type pollable</code></a></h4>
<p><code>u32</code></p>
<p>A "pollable" handle.
<p>This is conceptually represents a <code>stream&lt;_, _&gt;</code>, or in other words,
a stream that one can wait on, repeatedly, but which does not itself
produce any data. It's temporary scaffolding until component-model's
async features are ready.</p>
<p>And at present, it is a <code>u32</code> instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.</p>
<p><a href="#pollable"><code>pollable</code></a> lifetimes are not automatically managed. Users must ensure
that they do not outlive the resource they reference.</p>
<hr />
<h3>Functions</h3>
<h4><a name="poll_oneoff"><code>poll-oneoff: func</code></a></h4>
<p>Poll for completion on a set of pollables.</p>
<p>The &quot;oneoff&quot; in the name refers to the fact that this function must do a
linear scan through the entire list of subscriptions, which may be
inefficient if the number is large and the same subscriptions are used
many times. In the future, it may be accompanied by an API similar to
Linux's <code>epoll</code> which allows sets of subscriptions to be registered and
made efficiently reusable.</p>
<p>Note that the return type would ideally be <code>list&lt;bool&gt;</code>, but that would
be more difficult to polyfill given the current state of <code>wit-bindgen</code>.
See https://github.com/bytecodealliance/preview2-prototyping/pull/11#issuecomment-1329873061
for details.  For now, we use zero to mean &quot;not ready&quot; and non-zero to
mean &quot;ready&quot;.</p>
<h5>Params</h5>
<ul>
<li><a name="poll_oneoff.in"><code>in</code></a>: list&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="poll_oneoff.0"></a> list&lt;<code>u8</code>&gt;</li>
</ul>
<h2><a name="wasi_io">Import interface wasi-io</a></h2>
<p>WASI I/O is an I/O abstraction API which is currently focused on providing
stream types.</p>
<p>In the future, the component model is expected to add built-in stream types;
when it does, they are expected to subsume this API.</p>
<hr />
<h3>Types</h3>
<h4><a name="pollable"><code>type pollable</code></a></h4>
<p><a href="#pollable"><a href="#pollable"><code>pollable</code></a></a></p>
<p>
#### <a name="stream_error">`record stream-error`</a>
<p>An error type returned from a stream operation. Currently this
doesn't provide any additional information.</p>
<h5>Record Fields</h5>
<h4><a name="output_stream"><code>type output-stream</code></a></h4>
<p><code>u32</code></p>
<p>An output bytestream. In the future, this will be replaced by handle
types.
<p>This conceptually represents a <code>stream&lt;u8, _&gt;</code>. It's temporary
scaffolding until component-model's async features are ready.</p>
<p>And at present, it is a <code>u32</code> instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.</p>
<h4><a name="input_stream"><code>type input-stream</code></a></h4>
<p><code>u32</code></p>
<p>An input bytestream. In the future, this will be replaced by handle
types.
<p>This conceptually represents a <code>stream&lt;u8, _&gt;</code>. It's temporary
scaffolding until component-model's async features are ready.</p>
<p>And at present, it is a <code>u32</code> instead of being an actual handle, until
the wit-bindgen implementation of handles and resources is ready.</p>
<hr />
<h3>Functions</h3>
<h4><a name="read"><code>read: func</code></a></h4>
<p>Read bytes from a stream.</p>
<p>This function returns a list of bytes containing the data that was
read, along with a bool indicating whether the end of the stream
was reached. The returned list will contain up to <code>len</code> bytes; it
may return fewer than requested, but not more.</p>
<p>Once a stream has reached the end, subsequent calls to read or
<a href="#skip"><code>skip</code></a> will always report end-of-stream rather than producing more
data.</p>
<p>If <code>len</code> is 0, it represents a request to read 0 bytes, which should
always succeed, assuming the stream hasn't reached its end yet, and
return an empty list.</p>
<p>The len here is a <code>u64</code>, but some callees may not be able to allocate
a buffer as large as that would imply.
FIXME: describe what happens if allocation fails.</p>
<h5>Params</h5>
<ul>
<li><a name="read.this"><code>this</code></a>: <a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></li>
<li><a name="read.len"><code>len</code></a>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="read.0"></a> result&lt;(list&lt;<code>u8</code>&gt;, <code>bool</code>), <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="skip"><code>skip: func</code></a></h4>
<p>Skip bytes from a stream.</p>
<p>This is similar to the <a href="#read"><code>read</code></a> function, but avoids copying the
bytes into the instance.</p>
<p>Once a stream has reached the end, subsequent calls to read or
<a href="#skip"><code>skip</code></a> will always report end-of-stream rather than producing more
data.</p>
<p>This function returns the number of bytes skipped, along with a bool
indicating whether the end of the stream was reached. The returned
value will be at most <code>len</code>; it may be less.</p>
<h5>Params</h5>
<ul>
<li><a name="skip.this"><code>this</code></a>: <a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></li>
<li><a name="skip.len"><code>len</code></a>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="skip.0"></a> result&lt;(<code>u64</code>, <code>bool</code>), <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="subscribe_read"><code>subscribe-read: func</code></a></h4>
<p>Create a <a href="#pollable"><code>pollable</code></a> which will resolve once either the specified stream has bytes
available to read or the other end of the stream has been closed.</p>
<h5>Params</h5>
<ul>
<li><a name="subscribe_read.this"><code>this</code></a>: <a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="subscribe_read.0"></a> <a href="#pollable"><a href="#pollable"><code>pollable</code></a></a></li>
</ul>
<h4><a name="drop_input_stream"><code>drop-input-stream: func</code></a></h4>
<p>Dispose of the specified <a href="#input_stream"><code>input-stream</code></a>, after which it may no longer
be used.</p>
<h5>Params</h5>
<ul>
<li><a name="drop_input_stream.this"><code>this</code></a>: <a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></li>
</ul>
<h4><a name="write"><code>write: func</code></a></h4>
<p>Write bytes to a stream.</p>
<p>This function returns a <code>u64</code> indicating the number of bytes from
<code>buf</code> that were written; it may be less than the full list.</p>
<h5>Params</h5>
<ul>
<li><a name="write.this"><code>this</code></a>: <a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a></li>
<li><a name="write.buf"><code>buf</code></a>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="write.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="write_zeroes"><code>write-zeroes: func</code></a></h4>
<p>Write multiple zero bytes to a stream.</p>
<p>This function returns a <code>u64</code> indicating the number of zero bytes
that were written; it may be less than <code>len</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="write_zeroes.this"><code>this</code></a>: <a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a></li>
<li><a name="write_zeroes.len"><code>len</code></a>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="write_zeroes.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="splice"><code>splice: func</code></a></h4>
<p>Read from one stream and write to another.</p>
<p>This function returns the number of bytes transferred; it may be less
than <code>len</code>.</p>
<h5>Params</h5>
<ul>
<li><a name="splice.this"><code>this</code></a>: <a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a></li>
<li><a name="splice.src"><code>src</code></a>: <a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></li>
<li><a name="splice.len"><code>len</code></a>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="splice.0"></a> result&lt;(<code>u64</code>, <code>bool</code>), <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="subscribe"><code>subscribe: func</code></a></h4>
<p>Create a <a href="#pollable"><code>pollable</code></a> which will resolve once either the specified stream is ready
to accept bytes or the other end of the stream has been closed.</p>
<h5>Params</h5>
<ul>
<li><a name="subscribe.this"><code>this</code></a>: <a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="subscribe.0"></a> <a href="#pollable"><a href="#pollable"><code>pollable</code></a></a></li>
</ul>
<h4><a name="drop_output_stream"><code>drop-output-stream: func</code></a></h4>
<p>Dispose of the specified <a href="#output_stream"><code>output-stream</code></a>, after which it may no longer
be used.</p>
<h5>Params</h5>
<ul>
<li><a name="drop_output_stream.this"><code>this</code></a>: <a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a></li>
</ul>
<h2><a name="wasi_wall_clock">Import interface wasi-wall-clock</a></h2>
<p>WASI Wall Clock is a clock API intended to let users query the current
time. The name &quot;wall&quot; makes an analogy to a &quot;clock on the wall&quot;, which
is not necessarily monotonic as it may be reset.</p>
<p>It is intended to be portable at least between Unix-family platforms and
Windows.</p>
<hr />
<h3>Types</h3>
<h4><a name="wall_clock"><code>type wall-clock</code></a></h4>
<p><code>u32</code></p>
<p>A wall clock is a clock which measures the date and time according to some
external reference.
<p>External references may be reset, so this clock is not necessarily
monotonic, making it unsuitable for measuring elapsed time.</p>
<p>It is intended for reporting the current date and time for humans.</p>
<h4><a name="datetime"><code>record datetime</code></a></h4>
<p>A time and date in seconds plus nanoseconds.</p>
<h5>Record Fields</h5>
<ul>
<li><a name="datetime.seconds"><code>seconds</code></a>: <code>u64</code></li>
<li><a name="datetime.nanoseconds"><code>nanoseconds</code></a>: <code>u32</code></li>
</ul>
<hr />
<h3>Functions</h3>
<h4><a name="now"><code>now: func</code></a></h4>
<p>Read the current value of the clock.</p>
<p>This clock is not monotonic, therefore calling this function repeatedly will
not necessarily produce a sequence of non-decreasing values.</p>
<p>The returned timestamps represent the number of seconds since
1970-01-01T00:00:00Z, also known as <a href="https://pubs.opengroup.org/onlinepubs/9699919799/xrat/V4_xbd_chap04.html#tag_21_04_16">POSIX's Seconds Since the Epoch</a>, also
known as <a href="https://en.wikipedia.org/wiki/Unix_time">Unix Time</a>.</p>
<p>The nanoseconds field of the output is always less than 1000000000.</p>
<h5>Params</h5>
<ul>
<li><a name="now.this"><code>this</code></a>: <a href="#wall_clock"><a href="#wall_clock"><code>wall-clock</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="now.0"></a> <a href="#datetime"><a href="#datetime"><code>datetime</code></a></a></li>
</ul>
<h4><a name="resolution"><code>resolution: func</code></a></h4>
<p>Query the resolution of the clock.</p>
<p>The nanoseconds field of the output is always less than 1000000000.</p>
<h5>Params</h5>
<ul>
<li><a name="resolution.this"><code>this</code></a>: <a href="#wall_clock"><a href="#wall_clock"><code>wall-clock</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="resolution.0"></a> <a href="#datetime"><a href="#datetime"><code>datetime</code></a></a></li>
</ul>
<h4><a name="drop_wall_clock"><code>drop-wall-clock: func</code></a></h4>
<p>Dispose of the specified <a href="#wall_clock"><code>wall-clock</code></a>, after which it may no longer
be used.</p>
<h5>Params</h5>
<ul>
<li><a name="drop_wall_clock.this"><code>this</code></a>: <a href="#wall_clock"><a href="#wall_clock"><code>wall-clock</code></a></a></li>
</ul>
<h2><a name="wasi_filesystem">Import interface wasi-filesystem</a></h2>
<p>WASI filesystem is a filesystem API primarily intended to let users run WASI
programs that access their files on their existing filesystems, without
significant overhead.</p>
<p>It is intended to be roughly portable between Unix-family platforms and
Windows, though it does not hide many of the major differences.</p>
<p>Paths are passed as interface-type <code>string</code>s, meaning they must consist of
a sequence of Unicode Scalar Values (USVs). Some filesystems may contain paths
which are not accessible by this API.</p>
<hr />
<h3>Types</h3>
<h4><a name="input_stream"><code>type input-stream</code></a></h4>
<p><a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a></p>
<p>
#### <a name="output_stream">`type output-stream`</a>
[`output-stream`](#output_stream)
<p>
#### <a name="datetime">`type datetime`</a>
[`datetime`](#datetime)
<p>
#### <a name="o_flags">`flags o-flags`</a>
<p>Open flags used by <a href="#open_at"><code>open-at</code></a>.</p>
<h5>Flags members</h5>
<ul>
<li>
<p><a name="o_flags.create"><code>create</code></a>: </p>
<p>Create file if it does not exist.
</li>
<li>
<p><a name="o_flags.directory"><code>directory</code></a>: </p>
<p>Fail if not a directory.
</li>
<li>
<p><a name="o_flags.excl"><code>excl</code></a>: </p>
<p>Fail if file already exists.
</li>
<li>
<p><a name="o_flags.trunc"><code>trunc</code></a>: </p>
<p>Truncate file to size 0.
</li>
</ul>
<h4><a name="mode"><code>flags mode</code></a></h4>
<p>Permissions mode used by <a href="#open_at"><code>open-at</code></a>, <a href="#change_file_permissions_at"><code>change-file-permissions-at</code></a>, and
similar.</p>
<h5>Flags members</h5>
<ul>
<li>
<p><a name="mode.readable"><code>readable</code></a>: </p>
<p>True if the resource is considered readable by the containing
filesystem.
</li>
<li>
<p><a name="mode.writeable"><code>writeable</code></a>: </p>
<p>True if the resource is considered writeable by the containing
filesystem.
</li>
<li>
<p><a name="mode.executable"><code>executable</code></a>: </p>
<p>True if the resource is considered executable by the containing
filesystem. This does not apply to directories.
</li>
</ul>
<h4><a name="linkcount"><code>type linkcount</code></a></h4>
<p><code>u64</code></p>
<p>Number of hard links to an inode.
<h4><a name="inode"><code>type inode</code></a></h4>
<p><code>u64</code></p>
<p>Filesystem object serial number that is unique within its file system.
<h4><a name="filesize"><code>type filesize</code></a></h4>
<p><code>u64</code></p>
<p>File size or length of a region within a file.
<h4><a name="errno"><code>enum errno</code></a></h4>
<p>Error codes returned by functions.
Not all of these error codes are returned by the functions provided by this
API; some are used in higher-level library layers, and others are provided
merely for alignment with POSIX.</p>
<h5>Enum Cases</h5>
<ul>
<li>
<p><a name="errno.access"><code>access</code></a></p>
<p>Permission denied.
</li>
<li>
<p><a name="errno.again"><code>again</code></a></p>
<p>Resource unavailable, or operation would block.
</li>
<li>
<p><a name="errno.already"><code>already</code></a></p>
<p>Connection already in progress.
</li>
<li>
<p><a name="errno.badf"><code>badf</code></a></p>
<p>Bad descriptor.
</li>
<li>
<p><a name="errno.busy"><code>busy</code></a></p>
<p>Device or resource busy.
</li>
<li>
<p><a name="errno.deadlk"><code>deadlk</code></a></p>
<p>Resource deadlock would occur.
</li>
<li>
<p><a name="errno.dquot"><code>dquot</code></a></p>
<p>Storage quota exceeded.
</li>
<li>
<p><a name="errno.exist"><code>exist</code></a></p>
<p>File exists.
</li>
<li>
<p><a name="errno.fbig"><code>fbig</code></a></p>
<p>File too large.
</li>
<li>
<p><a name="errno.ilseq"><code>ilseq</code></a></p>
<p>Illegal byte sequence.
</li>
<li>
<p><a name="errno.inprogress"><code>inprogress</code></a></p>
<p>Operation in progress.
</li>
<li>
<p><a name="errno.intr"><code>intr</code></a></p>
<p>Interrupted function.
</li>
<li>
<p><a name="errno.inval"><code>inval</code></a></p>
<p>Invalid argument.
</li>
<li>
<p><a name="errno.io"><code>io</code></a></p>
<p>I/O error.
</li>
<li>
<p><a name="errno.isdir"><code>isdir</code></a></p>
<p>Is a directory.
</li>
<li>
<p><a name="errno.loop"><code>loop</code></a></p>
<p>Too many levels of symbolic links.
</li>
<li>
<p><a name="errno.mlink"><code>mlink</code></a></p>
<p>Too many links.
</li>
<li>
<p><a name="errno.msgsize"><code>msgsize</code></a></p>
<p>Message too large.
</li>
<li>
<p><a name="errno.nametoolong"><code>nametoolong</code></a></p>
<p>Filename too long.
</li>
<li>
<p><a name="errno.nodev"><code>nodev</code></a></p>
<p>No such device.
</li>
<li>
<p><a name="errno.noent"><code>noent</code></a></p>
<p>No such file or directory.
</li>
<li>
<p><a name="errno.nolck"><code>nolck</code></a></p>
<p>No locks available.
</li>
<li>
<p><a name="errno.nomem"><code>nomem</code></a></p>
<p>Not enough space.
</li>
<li>
<p><a name="errno.nospc"><code>nospc</code></a></p>
<p>No space left on device.
</li>
<li>
<p><a name="errno.nosys"><code>nosys</code></a></p>
<p>Function not supported.
</li>
<li>
<p><a name="errno.notdir"><code>notdir</code></a></p>
<p>Not a directory or a symbolic link to a directory.
</li>
<li>
<p><a name="errno.notempty"><code>notempty</code></a></p>
<p>Directory not empty.
</li>
<li>
<p><a name="errno.notrecoverable"><code>notrecoverable</code></a></p>
<p>State not recoverable.
</li>
<li>
<p><a name="errno.notsup"><code>notsup</code></a></p>
<p>Not supported, or operation not supported on socket.
</li>
<li>
<p><a name="errno.notty"><code>notty</code></a></p>
<p>Inappropriate I/O control operation.
</li>
<li>
<p><a name="errno.nxio"><code>nxio</code></a></p>
<p>No such device or address.
</li>
<li>
<p><a name="errno.overflow"><code>overflow</code></a></p>
<p>Value too large to be stored in data type.
</li>
<li>
<p><a name="errno.perm"><code>perm</code></a></p>
<p>Operation not permitted.
</li>
<li>
<p><a name="errno.pipe"><code>pipe</code></a></p>
<p>Broken pipe.
</li>
<li>
<p><a name="errno.rofs"><code>rofs</code></a></p>
<p>Read-only file system.
</li>
<li>
<p><a name="errno.spipe"><code>spipe</code></a></p>
<p>Invalid seek.
</li>
<li>
<p><a name="errno.txtbsy"><code>txtbsy</code></a></p>
<p>Text file busy.
</li>
<li>
<p><a name="errno.xdev"><code>xdev</code></a></p>
<p>Cross-device link.
</li>
</ul>
<h4><a name="dir_entry_stream"><code>type dir-entry-stream</code></a></h4>
<p><code>u32</code></p>
<p>A stream of directory entries.
<h4><a name="device"><code>type device</code></a></h4>
<p><code>u64</code></p>
<p>Identifier for a device containing a file system. Can be used in combination
with `inode` to uniquely identify a file or directory in the filesystem.
<h4><a name="descriptor_type"><code>enum descriptor-type</code></a></h4>
<p>The type of a filesystem object referenced by a descriptor.</p>
<p>Note: This was called <code>filetype</code> in earlier versions of WASI.</p>
<h5>Enum Cases</h5>
<ul>
<li>
<p><a name="descriptor_type.unknown"><code>unknown</code></a></p>
<p>The type of the descriptor or file is unknown or is different from
any of the other types specified.
</li>
<li>
<p><a name="descriptor_type.block_device"><code>block-device</code></a></p>
<p>The descriptor refers to a block device inode.
</li>
<li>
<p><a name="descriptor_type.character_device"><code>character-device</code></a></p>
<p>The descriptor refers to a character device inode.
</li>
<li>
<p><a name="descriptor_type.directory"><code>directory</code></a></p>
<p>The descriptor refers to a directory inode.
</li>
<li>
<p><a name="descriptor_type.fifo"><code>fifo</code></a></p>
<p>The descriptor refers to a named pipe.
</li>
<li>
<p><a name="descriptor_type.symbolic_link"><code>symbolic-link</code></a></p>
<p>The file refers to a symbolic link inode.
</li>
<li>
<p><a name="descriptor_type.regular_file"><code>regular-file</code></a></p>
<p>The descriptor refers to a regular file inode.
</li>
<li>
<p><a name="descriptor_type.socket"><code>socket</code></a></p>
<p>The descriptor refers to a socket.
</li>
</ul>
<h4><a name="dir_entry"><code>record dir-entry</code></a></h4>
<p>A directory entry.</p>
<h5>Record Fields</h5>
<ul>
<li>
<p><a name="dir_entry.ino"><code>ino</code></a>: option&lt;<a href="#inode"><a href="#inode"><code>inode</code></a></a>&gt;</p>
<p>The serial number of the object referred to by this directory entry.
May be none if the inode value is not known.
<p>When this is none, libc implementations might do an extra <a href="#stat_at"><code>stat-at</code></a>
call to retrieve the inode number to fill their <code>d_ino</code> fields, so
implementations which can set this to a non-none value should do so.</p>
</li>
<li>
<p><a name="dir_entry.type"><a href="#type"><code>type</code></a></a>: <a href="#descriptor_type"><a href="#descriptor_type"><code>descriptor-type</code></a></a></p>
<p>The type of the file referred to by this directory entry.
</li>
<li>
<p><a name="dir_entry.name"><code>name</code></a>: <code>string</code></p>
<p>The name of the object.
</li>
</ul>
<h4><a name="descriptor_flags"><code>flags descriptor-flags</code></a></h4>
<p>Descriptor flags.</p>
<p>Note: This was called <code>fdflags</code> in earlier versions of WASI.</p>
<h5>Flags members</h5>
<ul>
<li>
<p><a name="descriptor_flags.read"><a href="#read"><code>read</code></a></a>: </p>
<p>Read mode: Data can be read.
</li>
<li>
<p><a name="descriptor_flags.write"><a href="#write"><code>write</code></a></a>: </p>
<p>Write mode: Data can be written to.
</li>
<li>
<p><a name="descriptor_flags.nonblock"><code>nonblock</code></a>: </p>
<p>Requests non-blocking operation.
<p>When this flag is enabled, functions may return immediately with an
<a href="#errno.again"><code>errno::again</code></a> error code in situations where they would otherwise
block. However, this non-blocking behavior is not required.
Implementations are permitted to ignore this flag and block.</p>
</li>
<li>
<p><a name="descriptor_flags.sync"><a href="#sync"><code>sync</code></a></a>: </p>
<p>Request that writes be performed according to synchronized I/O file
integrity completion. The data stored in the file and the file's
metadata are synchronized.
<p>The precise semantics of this operation have not yet been defined for
WASI. At this time, it should be interpreted as a request, and not a
requirement.</p>
</li>
<li>
<p><a name="descriptor_flags.dsync"><code>dsync</code></a>: </p>
<p>Request that writes be performed according to synchronized I/O data
integrity completion. Only the data stored in the file is
synchronized.
<p>The precise semantics of this operation have not yet been defined for
WASI. At this time, it should be interpreted as a request, and not a
requirement.</p>
</li>
<li>
<p><a name="descriptor_flags.rsync"><code>rsync</code></a>: </p>
<p>Requests that reads be performed at the same level of integrety
requested for writes.
<p>The precise semantics of this operation have not yet been defined for
WASI. At this time, it should be interpreted as a request, and not a
requirement.</p>
</li>
<li>
<p><a name="descriptor_flags.mutate_directory"><code>mutate-directory</code></a>: </p>
<p>Mutating directories mode: Directory contents may be mutated.
<p>When this flag is unset on a descriptor, operations using the
descriptor which would create, rename, delete, modify the data or
metadata of filesystem objects, or obtain another handle which
would permit any of those, shall fail with <a href="#errno.rofs"><code>errno::rofs</code></a> if
they would otherwise succeed.</p>
<p>This may only be set on directories.</p>
</li>
</ul>
<h4><a name="descriptor"><code>type descriptor</code></a></h4>
<p><code>u32</code></p>
<p>A descriptor is a reference to a filesystem object, which may be a file,
directory, named pipe, special file, or other object on which filesystem
calls may be made.
<h4><a name="new_timestamp"><code>variant new-timestamp</code></a></h4>
<p>When setting a timestamp, this gives the value to set it to.</p>
<h5>Variant Cases</h5>
<ul>
<li>
<p><a name="new_timestamp.no_change"><code>no-change</code></a></p>
<p>Leave the timestamp set to its previous value.
</li>
<li>
<p><a name="new_timestamp.now"><a href="#now"><code>now</code></a></a></p>
<p>Set the timestamp to the current time of the system clock associated
with the filesystem.
</li>
<li>
<p><a name="new_timestamp.timestamp"><code>timestamp</code></a>: <a href="#datetime"><a href="#datetime"><code>datetime</code></a></a></p>
<p>Set the timestamp to the given value.
</li>
</ul>
<h4><a name="descriptor_stat"><code>record descriptor-stat</code></a></h4>
<p>File attributes.</p>
<p>Note: This was called <code>filestat</code> in earlier versions of WASI.</p>
<h5>Record Fields</h5>
<ul>
<li>
<p><a name="descriptor_stat.dev"><code>dev</code></a>: <a href="#device"><a href="#device"><code>device</code></a></a></p>
<p>Device ID of device containing the file.
</li>
<li>
<p><a name="descriptor_stat.ino"><code>ino</code></a>: <a href="#inode"><a href="#inode"><code>inode</code></a></a></p>
<p>File serial number.
</li>
<li>
<p><a name="descriptor_stat.type"><a href="#type"><code>type</code></a></a>: <a href="#descriptor_type"><a href="#descriptor_type"><code>descriptor-type</code></a></a></p>
<p>File type.
</li>
<li>
<p><a name="descriptor_stat.nlink"><code>nlink</code></a>: <a href="#linkcount"><a href="#linkcount"><code>linkcount</code></a></a></p>
<p>Number of hard links to the file.
</li>
<li>
<p><a name="descriptor_stat.size"><code>size</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></p>
<p>For regular files, the file size in bytes. For symbolic links, the length
in bytes of the pathname contained in the symbolic link.
</li>
<li>
<p><a name="descriptor_stat.atim"><code>atim</code></a>: <a href="#datetime"><a href="#datetime"><code>datetime</code></a></a></p>
<p>Last data access timestamp.
</li>
<li>
<p><a name="descriptor_stat.mtim"><code>mtim</code></a>: <a href="#datetime"><a href="#datetime"><code>datetime</code></a></a></p>
<p>Last data modification timestamp.
</li>
<li>
<p><a name="descriptor_stat.ctim"><code>ctim</code></a>: <a href="#datetime"><a href="#datetime"><code>datetime</code></a></a></p>
<p>Last file status change timestamp.
</li>
</ul>
<h4><a name="at_flags"><code>flags at-flags</code></a></h4>
<p>Flags determining the method of how paths are resolved.</p>
<h5>Flags members</h5>
<ul>
<li><a name="at_flags.symlink_follow"><code>symlink-follow</code></a>: <p>As long as the resolved path corresponds to a symbolic link, it is expanded.
</li>
</ul>
<h4><a name="advice"><code>enum advice</code></a></h4>
<p>File or memory access pattern advisory information.</p>
<h5>Enum Cases</h5>
<ul>
<li>
<p><a name="advice.normal"><code>normal</code></a></p>
<p>The application has no advice to give on its behavior with respect to the specified data.
</li>
<li>
<p><a name="advice.sequential"><code>sequential</code></a></p>
<p>The application expects to access the specified data sequentially from lower offsets to higher offsets.
</li>
<li>
<p><a name="advice.random"><code>random</code></a></p>
<p>The application expects to access the specified data in a random order.
</li>
<li>
<p><a name="advice.will_need"><code>will-need</code></a></p>
<p>The application expects to access the specified data in the near future.
</li>
<li>
<p><a name="advice.dont_need"><code>dont-need</code></a></p>
<p>The application expects that it will not access the specified data in the near future.
</li>
<li>
<p><a name="advice.no_reuse"><code>no-reuse</code></a></p>
<p>The application expects to access the specified data once and then not reuse it thereafter.
</li>
</ul>
<hr />
<h3>Functions</h3>
<h4><a name="read_via_stream"><code>read-via-stream: func</code></a></h4>
<p>Return a stream for reading from a file.</p>
<p>Note: This allows using <code>read-stream</code>, which is similar to <a href="#read"><code>read</code></a> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="read_via_stream.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="read_via_stream.offset"><code>offset</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="read_via_stream.0"></a> result&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="write_via_stream"><code>write-via-stream: func</code></a></h4>
<p>Return a stream for writing to a file.</p>
<p>Note: This allows using <code>write-stream</code>, which is similar to <a href="#write"><code>write</code></a> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="write_via_stream.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="write_via_stream.offset"><code>offset</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="write_via_stream.0"></a> result&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="append_via_stream"><code>append-via-stream: func</code></a></h4>
<p>Return a stream for appending to a file.</p>
<p>Note: This allows using <code>write-stream</code>, which is similar to <a href="#write"><code>write</code></a> with
<code>O_APPEND</code> in in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="append_via_stream.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="append_via_stream.fd"><code>fd</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="append_via_stream.0"></a> result&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="fadvise"><code>fadvise: func</code></a></h4>
<p>Provide file advisory information on a descriptor.</p>
<p>This is similar to <code>posix_fadvise</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="fadvise.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="fadvise.offset"><code>offset</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
<li><a name="fadvise.len"><code>len</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
<li><a name="fadvise.advice"><a href="#advice"><code>advice</code></a></a>: <a href="#advice"><a href="#advice"><code>advice</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="fadvise.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="datasync"><code>datasync: func</code></a></h4>
<p>Synchronize the data of a file to disk.</p>
<p>This function succeeds with no effect if the file descriptor is not
opened for writing.</p>
<p>Note: This is similar to <code>fdatasync</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="datasync.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="datasync.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="flags"><code>flags: func</code></a></h4>
<p>Get flags associated with a descriptor.</p>
<p>Note: This returns similar flags to <code>fcntl(fd, F_GETFL)</code> in POSIX.</p>
<p>Note: This returns the value that was the <code>fs_flags</code> value returned
from <code>fdstat_get</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="flags.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="flags.0"></a> result&lt;<a href="#descriptor_flags"><a href="#descriptor_flags"><code>descriptor-flags</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="type"><code>type: func</code></a></h4>
<p>Get the dynamic type of a descriptor.</p>
<p>Note: This returns the same value as the <a href="#type"><code>type</code></a> field of the <code>fd-stat</code>
returned by <a href="#stat"><code>stat</code></a>, <a href="#stat_at"><code>stat-at</code></a> and similar.</p>
<p>Note: This returns similar flags to the <code>st_mode &amp; S_IFMT</code> value provided
by <code>fstat</code> in POSIX.</p>
<p>Note: This returns the value that was the <code>fs_filetype</code> value returned
from <code>fdstat_get</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="type.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="type.0"></a> result&lt;<a href="#descriptor_type"><a href="#descriptor_type"><code>descriptor-type</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="set_flags"><code>set-flags: func</code></a></h4>
<p>Set status flags associated with a descriptor.</p>
<p>This function may only change the <code>append</code> and <code>nonblock</code> flags.</p>
<p>Note: This is similar to <code>fcntl(fd, F_SETFL, flags)</code> in POSIX.</p>
<p>Note: This was called <code>fd_fdstat_set_flags</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="set_flags.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="set_flags.flags"><a href="#flags"><code>flags</code></a></a>: <a href="#descriptor_flags"><a href="#descriptor_flags"><code>descriptor-flags</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="set_flags.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="set_size"><code>set-size: func</code></a></h4>
<p>Adjust the size of an open file. If this increases the file's size, the
extra bytes are filled with zeros.</p>
<p>Note: This was called <code>fd_filestat_set_size</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="set_size.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="set_size.size"><code>size</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="set_size.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="set_times"><code>set-times: func</code></a></h4>
<p>Adjust the timestamps of an open file or directory.</p>
<p>Note: This is similar to <code>futimens</code> in POSIX.</p>
<p>Note: This was called <code>fd_filestat_set_times</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="set_times.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="set_times.atim"><code>atim</code></a>: <a href="#new_timestamp"><a href="#new_timestamp"><code>new-timestamp</code></a></a></li>
<li><a name="set_times.mtim"><code>mtim</code></a>: <a href="#new_timestamp"><a href="#new_timestamp"><code>new-timestamp</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="set_times.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="pread"><code>pread: func</code></a></h4>
<p>Read from a descriptor, without using and updating the descriptor's offset.</p>
<p>Note: This is similar to <a href="#pread"><code>pread</code></a> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="pread.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="pread.len"><code>len</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
<li><a name="pread.offset"><code>offset</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="pread.0"></a> result&lt;list&lt;<code>u8</code>&gt;, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="pwrite"><code>pwrite: func</code></a></h4>
<p>Write to a descriptor, without using and updating the descriptor's offset.</p>
<p>It is valid to write past the end of a file; the file is extended to the
extent of the write, with bytes between the previous end and the start of
the write set to zero.</p>
<p>Note: This is similar to <a href="#pwrite"><code>pwrite</code></a> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="pwrite.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="pwrite.buf"><code>buf</code></a>: list&lt;<code>u8</code>&gt;</li>
<li><a name="pwrite.offset"><code>offset</code></a>: <a href="#filesize"><a href="#filesize"><code>filesize</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="pwrite.0"></a> result&lt;<a href="#filesize"><a href="#filesize"><code>filesize</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="readdir"><code>readdir: func</code></a></h4>
<p>Read directory entries from a directory.</p>
<p>On filesystems where directories contain entries referring to themselves
and their parents, often named <code>.</code> and <code>..</code> respectively, these entries
are omitted.</p>
<p>This always returns a new stream which starts at the beginning of the
directory.</p>
<h5>Params</h5>
<ul>
<li><a name="readdir.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="readdir.0"></a> result&lt;<a href="#dir_entry_stream"><a href="#dir_entry_stream"><code>dir-entry-stream</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="sync"><code>sync: func</code></a></h4>
<p>Synchronize the data and metadata of a file to disk.</p>
<p>This function succeeds with no effect if the file descriptor is not
opened for writing.</p>
<p>Note: This is similar to <code>fsync</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="sync.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="sync.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="create_directory_at"><code>create-directory-at: func</code></a></h4>
<p>Create a directory.</p>
<p>Note: This is similar to <code>mkdirat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="create_directory_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="create_directory_at.path"><code>path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="create_directory_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="stat"><code>stat: func</code></a></h4>
<p>Return the attributes of an open file or directory.</p>
<p>Note: This is similar to <code>fstat</code> in POSIX.</p>
<p>Note: This was called <code>fd_filestat_get</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="stat.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="stat.0"></a> result&lt;<a href="#descriptor_stat"><a href="#descriptor_stat"><code>descriptor-stat</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="stat_at"><code>stat-at: func</code></a></h4>
<p>Return the attributes of a file or directory.</p>
<p>Note: This is similar to <code>fstatat</code> in POSIX.</p>
<p>Note: This was called <code>path_filestat_get</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="stat_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="stat_at.at_flags"><a href="#at_flags"><code>at-flags</code></a></a>: <a href="#at_flags"><a href="#at_flags"><code>at-flags</code></a></a></li>
<li><a name="stat_at.path"><code>path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="stat_at.0"></a> result&lt;<a href="#descriptor_stat"><a href="#descriptor_stat"><code>descriptor-stat</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="set_times_at"><code>set-times-at: func</code></a></h4>
<p>Adjust the timestamps of a file or directory.</p>
<p>Note: This is similar to <code>utimensat</code> in POSIX.</p>
<p>Note: This was called <code>path_filestat_set_times</code> in earlier versions of WASI.</p>
<h5>Params</h5>
<ul>
<li><a name="set_times_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="set_times_at.at_flags"><a href="#at_flags"><code>at-flags</code></a></a>: <a href="#at_flags"><a href="#at_flags"><code>at-flags</code></a></a></li>
<li><a name="set_times_at.path"><code>path</code></a>: <code>string</code></li>
<li><a name="set_times_at.atim"><code>atim</code></a>: <a href="#new_timestamp"><a href="#new_timestamp"><code>new-timestamp</code></a></a></li>
<li><a name="set_times_at.mtim"><code>mtim</code></a>: <a href="#new_timestamp"><a href="#new_timestamp"><code>new-timestamp</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="set_times_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="link_at"><code>link-at: func</code></a></h4>
<p>Create a hard link.</p>
<p>Note: This is similar to <code>linkat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="link_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="link_at.old_at_flags"><code>old-at-flags</code></a>: <a href="#at_flags"><a href="#at_flags"><code>at-flags</code></a></a></li>
<li><a name="link_at.old_path"><code>old-path</code></a>: <code>string</code></li>
<li><a name="link_at.new_descriptor"><code>new-descriptor</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="link_at.new_path"><code>new-path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="link_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="open_at"><code>open-at: func</code></a></h4>
<p>Open a file or directory.</p>
<p>The returned descriptor is not guaranteed to be the lowest-numbered
descriptor not currently open/ it is randomized to prevent applications
from depending on making assumptions about indexes, since this is
error-prone in multi-threaded contexts. The returned descriptor is
guaranteed to be less than 2**31.</p>
<p>If <a href="#flags"><code>flags</code></a> contains <a href="#descriptor_flags.mutate_directory"><code>descriptor-flags::mutate-directory</code></a>, and the base
descriptor doesn't have <a href="#descriptor_flags.mutate_directory"><code>descriptor-flags::mutate-directory</code></a> set,
<a href="#open_at"><code>open-at</code></a> fails with <a href="#errno.rofs"><code>errno::rofs</code></a>.</p>
<p>If <a href="#flags"><code>flags</code></a> contains <a href="#write"><code>write</code></a> or <code>append</code>, or <a href="#o_flags"><code>o-flags</code></a> contains <code>trunc</code>
or <code>create</code>, and the base descriptor doesn't have
<a href="#descriptor_flags.mutate_directory"><code>descriptor-flags::mutate-directory</code></a> set, <a href="#open_at"><code>open-at</code></a> fails with
<a href="#errno.rofs"><code>errno::rofs</code></a>.</p>
<p>Note: This is similar to <code>openat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="open_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="open_at.at_flags"><a href="#at_flags"><code>at-flags</code></a></a>: <a href="#at_flags"><a href="#at_flags"><code>at-flags</code></a></a></li>
<li><a name="open_at.path"><code>path</code></a>: <code>string</code></li>
<li><a name="open_at.o_flags"><a href="#o_flags"><code>o-flags</code></a></a>: <a href="#o_flags"><a href="#o_flags"><code>o-flags</code></a></a></li>
<li><a name="open_at.flags"><a href="#flags"><code>flags</code></a></a>: <a href="#descriptor_flags"><a href="#descriptor_flags"><code>descriptor-flags</code></a></a></li>
<li><a name="open_at.mode"><a href="#mode"><code>mode</code></a></a>: <a href="#mode"><a href="#mode"><code>mode</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="open_at.0"></a> result&lt;<a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="readlink_at"><code>readlink-at: func</code></a></h4>
<p>Read the contents of a symbolic link.</p>
<p>Note: This is similar to <code>readlinkat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="readlink_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="readlink_at.path"><code>path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="readlink_at.0"></a> result&lt;<code>string</code>, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="remove_directory_at"><code>remove-directory-at: func</code></a></h4>
<p>Remove a directory.</p>
<p>Return <a href="#errno.notempty"><code>errno::notempty</code></a> if the directory is not empty.</p>
<p>Note: This is similar to <code>unlinkat(fd, path, AT_REMOVEDIR)</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="remove_directory_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="remove_directory_at.path"><code>path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="remove_directory_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="rename_at"><code>rename-at: func</code></a></h4>
<p>Rename a filesystem object.</p>
<p>Note: This is similar to <code>renameat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="rename_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="rename_at.old_path"><code>old-path</code></a>: <code>string</code></li>
<li><a name="rename_at.new_descriptor"><code>new-descriptor</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="rename_at.new_path"><code>new-path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="rename_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="symlink_at"><code>symlink-at: func</code></a></h4>
<p>Create a symbolic link.</p>
<p>Note: This is similar to <code>symlinkat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="symlink_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="symlink_at.old_path"><code>old-path</code></a>: <code>string</code></li>
<li><a name="symlink_at.new_path"><code>new-path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="symlink_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="unlink_file_at"><code>unlink-file-at: func</code></a></h4>
<p>Unlink a filesystem object that is not a directory.</p>
<p>Return <a href="#errno.isdir"><code>errno::isdir</code></a> if the path refers to a directory.
Note: This is similar to <code>unlinkat(fd, path, 0)</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="unlink_file_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="unlink_file_at.path"><code>path</code></a>: <code>string</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="unlink_file_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="change_file_permissions_at"><code>change-file-permissions-at: func</code></a></h4>
<p>Change the permissions of a filesystem object that is not a directory.</p>
<p>Note that the ultimate meanings of these permissions is
filesystem-specific.</p>
<p>Note: This is similar to <code>fchmodat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="change_file_permissions_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="change_file_permissions_at.at_flags"><a href="#at_flags"><code>at-flags</code></a></a>: <a href="#at_flags"><a href="#at_flags"><code>at-flags</code></a></a></li>
<li><a name="change_file_permissions_at.path"><code>path</code></a>: <code>string</code></li>
<li><a name="change_file_permissions_at.mode"><a href="#mode"><code>mode</code></a></a>: <a href="#mode"><a href="#mode"><code>mode</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="change_file_permissions_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="change_directory_permissions_at"><code>change-directory-permissions-at: func</code></a></h4>
<p>Change the permissions of a directory.</p>
<p>Note that the ultimate meanings of these permissions is
filesystem-specific.</p>
<p>Unlike in POSIX, the <code>executable</code> flag is not reinterpreted as a &quot;search&quot;
flag. <a href="#read"><code>read</code></a> on a directory implies readability and searchability, and
<code>execute</code> is not valid for directories.</p>
<p>Note: This is similar to <code>fchmodat</code> in POSIX.</p>
<h5>Params</h5>
<ul>
<li><a name="change_directory_permissions_at.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
<li><a name="change_directory_permissions_at.at_flags"><a href="#at_flags"><code>at-flags</code></a></a>: <a href="#at_flags"><a href="#at_flags"><code>at-flags</code></a></a></li>
<li><a name="change_directory_permissions_at.path"><code>path</code></a>: <code>string</code></li>
<li><a name="change_directory_permissions_at.mode"><a href="#mode"><code>mode</code></a></a>: <a href="#mode"><a href="#mode"><code>mode</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="change_directory_permissions_at.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="lock_shared"><code>lock-shared: func</code></a></h4>
<p>Request a shared advisory lock for an open file.</p>
<p>This requests a <em>shared</em> lock; more than one shared lock can be held for
a file at the same time.</p>
<p>If the open file has an exclusive lock, this function downgrades the lock
to a shared lock. If it has a shared lock, this function has no effect.</p>
<p>This requests an <em>advisory</em> lock, meaning that the file could be accessed
by other programs that don't hold the lock.</p>
<p>It is unspecified how shared locks interact with locks acquired by
non-WASI programs.</p>
<p>This function blocks until the lock can be acquired.</p>
<p>Not all filesystems support locking; on filesystems which don't support
locking, this function returns <a href="#errno.notsup"><code>errno::notsup</code></a>.</p>
<p>Note: This is similar to <code>flock(fd, LOCK_SH)</code> in Unix.</p>
<h5>Params</h5>
<ul>
<li><a name="lock_shared.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="lock_shared.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="lock_exclusive"><code>lock-exclusive: func</code></a></h4>
<p>Request an exclusive advisory lock for an open file.</p>
<p>This requests an <em>exclusive</em> lock; no other locks may be held for the
file while an exclusive lock is held.</p>
<p>If the open file has a shared lock and there are no exclusive locks held
for the file, this function upgrades the lock to an exclusive lock. If the
open file already has an exclusive lock, this function has no effect.</p>
<p>This requests an <em>advisory</em> lock, meaning that the file could be accessed
by other programs that don't hold the lock.</p>
<p>It is unspecified whether this function succeeds if the file descriptor
is not opened for writing. It is unspecified how exclusive locks interact
with locks acquired by non-WASI programs.</p>
<p>This function blocks until the lock can be acquired.</p>
<p>Not all filesystems support locking; on filesystems which don't support
locking, this function returns <a href="#errno.notsup"><code>errno::notsup</code></a>.</p>
<p>Note: This is similar to <code>flock(fd, LOCK_EX)</code> in Unix.</p>
<h5>Params</h5>
<ul>
<li><a name="lock_exclusive.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="lock_exclusive.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="try_lock_shared"><code>try-lock-shared: func</code></a></h4>
<p>Request a shared advisory lock for an open file.</p>
<p>This requests a <em>shared</em> lock; more than one shared lock can be held for
a file at the same time.</p>
<p>If the open file has an exclusive lock, this function downgrades the lock
to a shared lock. If it has a shared lock, this function has no effect.</p>
<p>This requests an <em>advisory</em> lock, meaning that the file could be accessed
by other programs that don't hold the lock.</p>
<p>It is unspecified how shared locks interact with locks acquired by
non-WASI programs.</p>
<p>This function returns <a href="#errno.again"><code>errno::again</code></a> if the lock cannot be acquired.</p>
<p>Not all filesystems support locking; on filesystems which don't support
locking, this function returns <a href="#errno.notsup"><code>errno::notsup</code></a>.</p>
<p>Note: This is similar to <code>flock(fd, LOCK_SH | LOCK_NB)</code> in Unix.</p>
<h5>Params</h5>
<ul>
<li><a name="try_lock_shared.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="try_lock_shared.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="try_lock_exclusive"><code>try-lock-exclusive: func</code></a></h4>
<p>Request an exclusive advisory lock for an open file.</p>
<p>This requests an <em>exclusive</em> lock; no other locks may be held for the
file while an exclusive lock is held.</p>
<p>If the open file has a shared lock and there are no exclusive locks held
for the file, this function upgrades the lock to an exclusive lock. If the
open file already has an exclusive lock, this function has no effect.</p>
<p>This requests an <em>advisory</em> lock, meaning that the file could be accessed
by other programs that don't hold the lock.</p>
<p>It is unspecified whether this function succeeds if the file descriptor
is not opened for writing. It is unspecified how exclusive locks interact
with locks acquired by non-WASI programs.</p>
<p>This function returns <a href="#errno.again"><code>errno::again</code></a> if the lock cannot be acquired.</p>
<p>Not all filesystems support locking; on filesystems which don't support
locking, this function returns <a href="#errno.notsup"><code>errno::notsup</code></a>.</p>
<p>Note: This is similar to <code>flock(fd, LOCK_EX | LOCK_NB)</code> in Unix.</p>
<h5>Params</h5>
<ul>
<li><a name="try_lock_exclusive.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="try_lock_exclusive.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="unlock"><code>unlock: func</code></a></h4>
<p>Release a shared or exclusive lock on an open file.</p>
<p>Note: This is similar to <code>flock(fd, LOCK_UN)</code> in Unix.</p>
<h5>Params</h5>
<ul>
<li><a name="unlock.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="unlock.0"></a> result&lt;_, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="drop_descriptor"><code>drop-descriptor: func</code></a></h4>
<p>Dispose of the specified <a href="#descriptor"><code>descriptor</code></a>, after which it may no longer
be used.</p>
<h5>Params</h5>
<ul>
<li><a name="drop_descriptor.this"><code>this</code></a>: <a href="#descriptor"><a href="#descriptor"><code>descriptor</code></a></a></li>
</ul>
<h4><a name="read_dir_entry"><code>read-dir-entry: func</code></a></h4>
<p>Read a single directory entry from a <a href="#dir_entry_stream"><code>dir-entry-stream</code></a>.</p>
<h5>Params</h5>
<ul>
<li><a name="read_dir_entry.this"><code>this</code></a>: <a href="#dir_entry_stream"><a href="#dir_entry_stream"><code>dir-entry-stream</code></a></a></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="read_dir_entry.0"></a> result&lt;option&lt;<a href="#dir_entry"><a href="#dir_entry"><code>dir-entry</code></a></a>&gt;, <a href="#errno"><a href="#errno"><code>errno</code></a></a>&gt;</li>
</ul>
<h4><a name="drop_dir_entry_stream"><code>drop-dir-entry-stream: func</code></a></h4>
<p>Dispose of the specified <a href="#dir_entry_stream"><code>dir-entry-stream</code></a>, after which it may no longer
be used.</p>
<h5>Params</h5>
<ul>
<li><a name="drop_dir_entry_stream.this"><code>this</code></a>: <a href="#dir_entry_stream"><a href="#dir_entry_stream"><code>dir-entry-stream</code></a></a></li>
</ul>
