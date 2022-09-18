---
layout: post
title:  "Virtual File Systems in Games!"
date:   2019-03-20 15:28:52 -0700
categories: architecture
---
After a particularly elucidating conversation with [Mattias Gustavsson on Twitter](https://web.archive.org/web/20190320091637/https://twitter.com/Mattias_G), I came to the delightful realization that a well utilized virtual path system can be make writing games a lot more fun. This blog post describes some basic, but strong, benefits in favor of using a virtual file system, as opposed to using the OS’s native file system alone.

---

### Ranting about (poor) Virtual File Systems

If readers are at all like myself, they may have tinkered with some larger engines that utilized virtual file paths all over the place. Often at work I would have to use the debugger and enter into this kind of virtual file system to figure out where things are located on disk, or where they are supposed to be located on disk. Typically this was a huge hassle, since the systems I used at the time were: A) not documented at all; B) not understood by anyone currently at the company; and C) over-engineered with incredibly deep and superfluous callstacks.

My initial experience with these systems was quite horrible, so I was very biased against them for a number of years. However, if used and written properly, some important benefits can come about from virtual file systems.

### What is a Virtual File System?

Typically the kind of virtual file system I have encountered somehow translate actual file paths on disk to a virtualized file path of some kind. Usually the system will truncate off the disk drive letters on Windows, and pretend the root of paths is located at some directory. Here is an example:

Say your game’s executable is located at C:/Program Files/Game/game.exe, and needs to open two other folders:

1. C:/Program Files/Game/data
2. C:/Users/Bob/Game/save_files

The first folder is where all game assets are stored on disk, like the art, animations, music, etc. The second folder is where the game writes out player save information, like their character or level save states.

Hard-coding these paths will not work if the game needs to run on platforms other than Windows. One nice way to gracefully handle multiple platforms is to use a virtual file system that can mount folders under an alias. Look at this code example:

{% highlight cpp %}
// vfs - Virtual File System
// A hypothetical library implementing useful file-related features.
#include <vfs.h>

int main()
{
	vfs_t* vfs = vfs_create();

	const char* dir_on_disk = "./data";
	const char* new_alias = "/data";
	vfs_mount(vfs, ".", new_alias);

	void* data;
	int size;
	if (vfs_read_file_to_memory(vfs, "/data/boat.png", &data, &size) < 0) {
		printf("Unable to find boat file.");
		return -1;
	}

	// On windows, for example, can be something like:
	// C:/Users/Bob/Game/saves
	const char* virtual_save_folder = get_virtual_save_folder();
	vfs_mount(vfs, virtual_save_folder, "/saves");

	// Open up save.txt, which resides on disk at C:/Users/Bob/Game/saves/save.txt
	vfs_file_t* fp = vfs_open_file(vfs, "/saves/save.txt");
	if (!fp) {
		printf("Unable to open save file.");
		return -1;
	}

	// Now save the game!
	save_the_game(vfs, fp, get_game_data());
}
{% endhighlight %}

The nice benefit here is that now game code can focus on local directories without needing to know specifically where on the disk the local directory resides. This removes the underlying OS characteristics from the game as dependencies, in trade for the virtual file system dependency.

Assuming the virtual file system is well implemented and used properly (which is often not the case in practice!), the tradeoff can be really good.

### Two Major Benefits

My #1 favorite benefit of mounting archives as virtual folders. My #2 favorite benefit is outlined by the above section: removing OS file-system characteristics from as much of the game as possible, in an attempt to make the game easier to port to multiple platforms.

I will focus on #1. The above section mounted actual directories under a virtual alias. It is also great if the virtual file system can mount archives under an alias, while still allowing file read operations as-per usual. This can grant the game the ability to read from archives or folders without requiring any changes to code whatsoever.

One huge benefit here is ease of distribution of the game. During development the majority of assets can sit on disk in separate files, making them easy to modify as needed. When ship time comes, the folder can be archived (example: zipped up as a .zip file), and the game will still run seamlessly.

Another benefit here is run-time efficiency. For example, on Windows opening a file is itself a heavy-weight operation. I am not exactly sure why this is the case, but my cursory research says it seems mostly slow due to security checks on file before opening (if any affluent readers could clarify, please post a comment or shoot me an email!). However, if many assets sit in a single archive, going through the initial “open the file handle” phase only has to happen once. This can be a giant time saver, just in terms of opening many individual files.

### Extensions, Patches, or Mods

If the virtual file system can handle duplicate aliases, then some nice benefits pop out for adding on changes or extensions to game assets. A good example is applying patches to a game. One way to patch a game is to modify old archives on-disk, but another strategy is to simply add another archive entirely.

If another archive can be added to the game’s directory, but mounted under the same alias as the original archive, an interesting effect takes place. Say we have two archives, A and B. Say A is the original, and B applies some patch (it has updated contents for only a small section of A). If the game first mounts A, and then mounts B under the same alias, then we can think about the search-path of the virtual file system.

Say we try to read a file from the archive. The virtual file system will search through its internal virtual file path and report the first match it finds. If the path is constructed in a known order, it can be guaranteed that the patch archive B will be considered before the original archive A. No game code needs to change at all, and only a new archive needs to be added (assuming the game looks for patches to mount when booted up).

Once another patch gets rolled out, C, it can be mounted in front of A and B in the virtual path! And so on, and so forth. Adding in new archives can be a very easy way for customers to modify the art assets of your game to their liking, and also be a great way to apply small patches to your game.

Obviously large-scale and numerous patches would probably be best done by modifying the original archive… But for smaller and less frequent changes, dropping in a new archive file is very simple and cheap!

### Other Benefits

I’m sure there are lots of other benefits that require a bit more in-depth knowledge than I have (for example, dealing with game-console oddities), so if anyone has any other ideas please do post them in the comments!

### Some Recommended Virtual File Systems

I have experience with two different virtual file systems, and can recommend them both.

* [assetsys by Mattias Gustavsson](https://web.archive.org/web/20190320091637/https://github.com/mattiasgustavsson/libs)
* [PhysicsFS by Icculus](https://web.archive.org/web/20190320091637/https://icculus.org/physfs/)

assetsys is the easier of the two to get up and running, but has less features than PhysicsFS. I would recommend trying assetsys if you’re looking for a single-file-header sort of drop-in solution, with a very small and focused API.

PhysicsFS is a more heavy-duty and serious solution that supports lots of platforms, and has a lot more features. This library will be a bit harder to integrate into pre-existing projects, but with cmake isn’t too bad to build from source.

Both libraries are very well written! I’ve used both in my personal code, and more recently am leaning towards PhysicsFS since it has a long history of very active development.
