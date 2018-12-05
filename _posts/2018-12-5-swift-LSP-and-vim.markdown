---
layout: post
title: Neovim and Swift, a Match Made in Heaven
---
<img src="/images/fulls/justin-greene-137326-unsplash.jpg" class="fit image">
<em>Photo by Justin Greene on Unsplash</em>

Every once in a while two technologies will converge, yielding a sum that is greater than their parts. If you are lucky enough, you will witness just such an event once or maybe twice in your life. Swift LSP combined with Neovim is one of these cataclysmic events. Hyperbole aside, I'm really excited about the prospect of effectively writing Swift outside of XCode. Once the project was announced, I couldn't wait to dive in and integrate it with my favorite editor.

#### What is Swift LSP?
Recently the Swift team announced that they would be releasing an LSP server (Language Server Protocol) for Swift. An LSP server, developed for a particular language, allows any editor which implements an LSP client to attain many IDE like features for editing that language, for free. These features include code completion, formatting, refactoring, go to definition, find references, contextual documentation look up and much more.

Before LSP, each IDE or editor had to implement all the above features for each language they supported. As a result, most editors would have a patchwork of functionality implemented for the few languages they supported. Polyglot developers would have to learn a new IDE for each language they used or do without advanced editing features.

While the Swift team has made amazing progress on the LSP server, it is still an immature project. Currently the only supported LSP features are code completion, go to definition, documentation lookup and find all references. Even though this is only a small subset of what LSP has to offer, it is already a big boost to developer productivity.

### How it works
Before I get into how to integrate the server with Neovim, I need to talk a bit about how an LSP server works. An LSP server needs to know how all the source files in a project relate to each other. Therefore, the server must know something about how the project is built. It needs to know which files are included in the project, where to find dependencies and which flags are passed to the compiler. The Swift LSP uses the Swift Package Manager to understand how the project is built. As such, if you want to use Swift LSP, your project needs be a Swift package. I wont go into detail on creating a package but you will need a Package.swift at the root of your project. 

You will also need to install the Swift tool chain describe on the Swift LSP [website](https://github.com/apple/sourcekit-lsp). You will then need to define `SOURCEKIT_TOOLCHAIN_PATH` in your `.zshrc`. It will look something like `export SOURCEKIT_TOOLCHAIN_PATH=<path to toolchain>`

Additionally, we will need an LSP Client. In our case, we will use [LanguageClient-neovim](https://github.com/autozimu/LanguageClient-neovim) (LCN form here on out), a Neovim plugin. Not only will this give you IDE like features for Swift but also for any other language implementing an LSP server.

I've created a demo repository that handles installing all the required Neovim plugins, downloads and build the LSP server and implements a small Swift package to experiment with. It can be found [here](https://github.com/Tylerc230/SwiftNeoVimLSP). It uses the Dein plugin manager for Neovim to install plugins. It also uses Deoplete for code completion. LCN will fallback to use omniFunc to do completion so Deoplete isn't strictly required. The project uses its own vimrc and puts all plugins in the `cache` folder so your Neovim installation should not be affected.

### How to use the demo repo
To use the repository, there are 3 steps:
1. Clone [my repo](https://github.com/Tylerc230/SwiftNeoVimLSP), cd into it and `./init.sh`.
This will fetch the `sourcekit-lsp` submodule, build it and build the demo Swift package found at `MySwiftPackage`. It will also update neovim with the Dein, Deoplete and LCN plugins. 
2. Download and install the Swift toolchain recommended on the [sourcekit-lsp website](https://github.com/apple/sourcekit-lsp)([12-4-2018](https://swift.org/builds/development/xcode/swift-DEVELOPMENT-SNAPSHOT-2018-12-04-a/swift-DEVELOPMENT-SNAPSHOT-2018-12-04-a-osx.pkg) at the time of this writing). Make sure `SOURCEKIT_TOOLCHAIN_PATH` in `tryit.sh` points to the newly installed xctoochain file.
3. Run `tryit.sh` which will set some environment variables and launch Neovim into the `main.swift` file. At this point you are ready to start experimenting!
3. In Neovim, run :UpdateRemotePlugins

### Things to try
1. If you move the cursor over a symbol eg. `MyStruct` and type `gd` in normal mode. You will be taken to the `MyStruct` definition in the `Greeter.swift` file. `gd` is mapped to `:call LanguageClient#textDocument_definition()<CR>`
2. Open a line under line 4 and type `my_array.`. You should see a list of all methods and properties defined by `Array`. You should be able to Tab through all the options.
3. Move the cursor over `first` in `my_array.first!` and type `:call LanguageClient#textDocument_hover()<CR>`. This should open up a preview window with the documentation for `Array.first` property.
4. Type `:call LanguageClient_contextMenu()<CR>`. This will bring up a context menu allowing you to select from the actions above. Most options are not implemented yet but gives you an idea of what will be possible in the future.
5. The Swift LSP server docs say that find all references is supported but I wasn't able to make it work. If you figure it out, drop me a line.

As you can see, the functionality is currently somewhat limited but the Swift team is moving fast. Even as I wrote this new commits were being pushed. Neovim is a pleasure to use and with the advent of the Swift LSP server I could see my self moving to it for all my Swift development needs.

### Caveats
There are a few catches that I ran into. First of all, you need to tell LCN how to find the root of the Swift project. In the vimrc file I told LCN to look for a file called `Package.swift` which indicates the root of a Swift project. By default LCN looks for a `.git` folder which, in our case, is one level up from our SPM root.
Swift LSP also doesn't index in the background, it will only index a project's symbols at build time. This means that the LSP server wont be aware of any new symbols added to a project until it is recompiled.
The Swift LSP project is moving fast so make sure you always have the latest versions of everything.
