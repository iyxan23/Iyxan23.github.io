----
title: My Newbie Experience with Neovim
author: NurIhsan
date: 2024-04-18 21:17:32 +0700
tags: vim journal for-fun
categories: Journals
----

Hi all! Welcome to another blog post that I wrote. Today I’ll be sharing a document I recently wrote on my journal. Reading it over, I feel like it could be published as a blog, so I did.

In this blog, I’m going to outline my experience with AstroNvim. Considering there are a few resources online, so I thought I’d put this here in case someone finds it useful.

It may not be accurate, because it literally is a journal. It’s not a technical document or anything. It’s just me having fun (and getting distracted from my actual project lol).

Anyways, have fun!

______________________________________________________________________

VSCode has been bugging me for a while for its lack of performance consideration put in to its development.

I had this AstroNvim alternative for a while just in case when things turned south. And it did.

So today, I decided to commit using AstroNvim, and decided to re-learn it again. And I somehow became a bit addicted into configuring it now!

It’s so cool! You can configure pretty much everything using `.lua` code. And of course a lot of crazy people did the work for us.

It can have LSPs, completions, even codeium and github copilot works awesome!

I had always wanted to migrate to nvim but it has a large learning curve, I need to understand how it works to effectively use it. I relied a bit too much on AstroNvim’s pre-configuration, which I find the experience awesome, but if you don’t know what its doing, its kinda hard to deal with problems that may arise with the configuration.

Now I had discovered quite a lot, I pretty much spent this whole day dedicating to migrating myself to neovim, and it’s probably the coolest and the best thing I did.

# Integration with Lua

In Vim, when you want to create a globally-scoped variable, you use a `g:` prefix before the variable name. So `set g:my_var = "hello"`.

I think this also reflects with neovim’s lua (neovim or nvim for short is a superset of vim that introduces lua as a second option to configure it).

Neovim exposes an API in a variable named `vim` (I think VimScript also does this, but idk). There, you are given a lot of APIs to work with with VIM, pretty much control all of it.

The one with `g` can be accessed with `vim.g.something`, quite the same as how you do with VimScript.

I also learned that lua modules (well atleast how neovim did it) are managed through nesting lua files inside nested folders. Similar to how python works, you need to have an `init.lua` file in a folder (on python that’d be `__init__.py`).

Exporting things also couldn’t be easier, you just do a top-level return statemet inside the code.

# Plugin System

How neovim works is very basic, it just runs some lua file (I’m not sure which or how many exactly) contained within the `~/.config/nvim` directory.

Sometimes you’d want to use other people’s things, so a plugin system is developed by the community to keep track (installing, updating, uninstalling) of plugins by leveraging the power of git.

There are dozens of plugins systems for neovim, but they function pretty much the same, just that they may provide different ways of doing it.

A plugin system could be considered as just a git wrapper but for neovim plugins. It clones repositories from github, then “sources” (like how shells do it) them within neovim. Very simple.

AstroNvim uses a plugin system named `lazy.nvim`. It’s known for its ability to lazy-load plugins, but honestly I don’t know what that means, I might try to explore a bit more.

AstroNvim is a pre-configured configuration of nvim that adds a lot of things you’d find on vscode. It does things like completion with LSP, formatting, and a lot of other things you might not find in VScode!

It’s composed of many community-plugins, you could say it is a glue that connects all of them together to form the best text editor in the world!

To define your own plugin or override astro’s defined plugins, you go to the folder `~/.config/nvim/lua/plugins`, create a new lua file and do a top-level return:

```lua
return {
  "github-user/my-plugin",
  opts = function (idk, opts)
    -- do a lot of config here
  end
}
```

It’s that easy!

If you have something that might need to be wired into AstroNvim, you might need to do quite some trickery to do it.

# Building Blocks of Vim: `Buffer`s & `AutoCmd`s

I learned these through trial and errors 🙃

I’m too lazy to read the docs of vim because they tend to cover too much (honestly because the editor itself is crazy complex, it has way too many features, yet being super lightweight).

So in vim, there are things that are called “buffers”.

Buffers are essentially the text files that you open in your vim editor. You can switch between different buffers (files) without leaving the editor. It's like having multiple tabs open in a web browser, but inside your text editor!

Buffers can also be compared to a “panel”, where a plugin may use to display things inside them. Like, the vertical file explorer is made by utilizing a buffer, but rather than displaying a file’s contents, it displays the filesystem tree and responds to mouse actions differently. It can also make its own commands, (like if you press the key `a` on the file explorer buffer, it will create a new file).

It can also have properties, like the `modifiable` property. When you are on a buffer that can’t be edited and you try to get into `INSERT` mode, it will fail, saying that the buffer is not `modifiable`. With buffers, you can get really crazy with anything!

Statusline and the tab displayer is also implemented this way, which is really cool.

In the internals of vim, it’s like an event-driven system where some parts of the code fire events, and some do something with those events. These events are called `AutoCmds`.

There’s an autocmd named `TermOpen` if you have `termtoggle.nvim` plugin installed (actually it is built-in into nvim itself)

`TermOpen` is dispatched when a terminal is opened. And what I did to my config is that I set some keybindings to the terminal.

Here’s a piece:

```lua
function _G.set_terminal_keymaps()
  local opts = {buffer = 0}
  
  -- lots of vim.keymap calls
end

vim.cmd('autocmd! TermOpen term://*toggleterm#1 lua set_terminal_keymaps()')
```

Reading the help manual, I had just realised that `autocmd`s is the literal building blocks of vim!

It has events like `BufFileNew`, `BufNew`, `CmdlineEnter`, and even `CursorHold`, proving the fact that everything is done by autocmds! Isn’t that really cool!?

# Realising I have an old version of AstroNvim

I tried to follow a guide on neovim’s website, only to get errors on my screen. It weirdly didn’t work, considering that it came from neovim’s website directly, and it gave a cryptic error that I cannot understand.

I had scoured quite a lot of websites and plugins to find things that I could incorporate to my neovim configuration. And some plugins describe themeslves to only be compatible with astronvim v4+, which I find quite interesting.

They explicitly made the plugin for astro v4+, as if there’s a big breaking change which made the plugin obsolete if its v3 or under.

So I checked my astronvim version, which apparently is v3.4.something, very far behind the latest version.

When I tried to upgrade it (since astro’s config is literally a git repo, I just did a basic git command of `git fetch`), it is exactly 261 commits behind latest. So I pulled to the latest, only to find out my nvim is completely bricked.

When I try opening nvim, it displays a message saying that “this configuration should’nt be used directly”, and when I press enter, I got kicked out of nvim.

Weird.

I searched about migrating to v4, [read the page](https://docs.astronvim.com/configuration/v4_migration/), and thought, wow that’s a really big architectural change.

But before we get into what that change is, we must understand how astro is installed.

## How AstroNvim v3 is Installed

Neovim reads files in `~/.config/nvim/` and just executes them.

This is the entrypoint of everything configurable in nvim, so astro is installed by cloning astro the repository into that directory. Since its a git repository, you can update by just `git pull`, which is quite cool.

To configure astro further, you need to create a user folder inside it, and create lua files according to astro’s standards. It’s similar to how NextJS uses the `page.tsx`, `layout.tsx`, etc convention. See astro’s ≤v3 user config [here](https://github.com/AstroNvim/user_example).

The structure more or less looks like this:

```
~/.config/nvim
L ...astro's files...
L init.lua         <- astro-managed file
L user             <- you make this folder
  L plugins        <- where you put your plugins
    L hop.lua      <- manages hop.nvim plugin
  L init.lua       <- what you're going to do
  L mappings.lua   <- your own mappings
  L options.lua    <- astro things you wanna tweak
```

Here’s how `mappings.lua` look like:

```lua
return {
  n = {
    ["<leader><leader>w"] = {
      function ()
	      -- do things here
	      require("hop").hopWordsAhead()
      end,
      desc = "Hop words ahead"
    },
    -- and more...
  }
}
```

With this, you can configure your own keybindings, and use lua to determine what to do.

You can create a new tab, open a new terminal, modify the buffer, even do some network stuff, heck your imagination is the limit!

The same thing applies to `options.lua`, but you put configuration that you want to apply to astro-managed plugins. You get the gist.

## The Architectural Change of v4

The problem is that, since AstroNvim is installed this way, it is arguably somehow unmodular. So the AstroNvim thought of separating these into its own plugins, with the name `astro` prefixed to them!

As you can read on this page: [Migration to v4.0](https://docs.astronvim.com/configuration/v4_migration/)

They segregated (i think that’s how the word is supposed to be used) the many options and configuration into multiple astro plugins:

- `astroui`
- `astrocore`
- `astrolsp`
- `AstroNvim`

I honestly don’t fully understand what these plugins control or cover, but they are the “broken down” plugins of the previously monolithic AstroNvim codebase.

As seen from AstroNvim’s migration guide page, they moved several configuration to their respective plugins. It is quite a task, but not really difficult, most of the things are done as expected, and you don’t need to scour the codebase to much.

What I did was I migrated them, and got it up and running!

[Here’s my v4 config.](https://github.com/iyxan23/astronvim-user) And for comparison, [here’s my v3 config](https://github.com/iyxan23/astronvim-conf).

# Favourite Things on AstroNvim

Things that I absolutely LOVE using astro:

- `<leader>fw` - live grep of every files by telescope
- `<leader>fo` - quick open last opened files by telescope
- `<leader>ff` - find files quickly by telescope
- `ci(`, `ci{`, `ca<`, etc - ability to quickly modify things inside or around parentheses, brackets, tags, all the things!
- `hop.nvim` - moving around a file is SO MUCH BETTER, you don’t need to use your mouse
- `:s/change/change this/gcm` - ability to substitute quickly had blown my minds
- `gr`, `<leader>lr`, etc - using LSP VERY QUICKLY
- `{` `}` - moving between paragraphs is a breeze
- `dd` or `yy` - quickly modify lines, like really
- it’s FREAKING FAST, of course it is! It’s a TUI compared to a fat electron app.
