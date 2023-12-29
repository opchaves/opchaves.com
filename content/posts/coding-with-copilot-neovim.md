---
title: "Coding With Copilot on Neovim + AstroNvim"
tags: ["neovim", "ai", "git"]
date: 2023-12-29T12:04:27
---

### Introduction

I have been using neovim for a long time and it is indeed my favorite text editor so far.
There is another one that I really like and is really popular these days, it's called VSCode.
I have thought about switching to VSCode to write code fulltime but I am unable since I cannot
exit Vim. I feel much more confortable writing code with neovim on the terminal with tmux to
manage multiple windows than on VSCode and I have tried using some vim plugin on VSCode that
would allow me to take advantage of vim features but I did not really like the experience.

Let's change the topic a bit and move to artificial intelligence (AI).
Nowadays AI is a big thing in the tech industry and everyone is talking about it.
Many startups are begin created and building products on top of ChatGPT for example.
Microsoft has an AI product that aims to make programmers more productive by using that tool.
The tool is called GitHub Copilot.

> GitHub Copilot is an AI pair programmer. You can use GitHub Copilot to get suggestions
> for whole lines or entire functions right inside your editor.
>
> <cite>[GitHub Docs][1]</cite>

Recently I managed to get a Copilot licence and finally I entered the AI world and can
now write code with AI giving me suggestions along the way and one thing I have to admit,
it is really cool and I feel more productive with it.

You can use Copilot in your preferred environment. I have Copilot setup on neovim+AstroNvim.
Setting up neovim from scratch to develop applications with Nodejs and Reactjs, which is my case,
is a daunting task and since I did not want to spend much time doing that by hand I went to the market
to look for a config that would give me things like autocomplete, custom fonts, fuzzy file search,
syntax highlight, git support and so on. I was lucky to find [AstroNvim](https://docs.astronvim.com/).
It is a tool that I use daily and since I added the Copilot plugin to it things got a lot
better for me. Enough talk, let me show you how I did it.

### Setting Up AstroNvim

[To get AstroNvim up an running](https://docs.astronvim.com/) all you need to do is backup your current `nvim` config and then clone
the AstroNvim repo:

```sh
# backup
mv ~/.config/nvim ~/.config/nvim.bak
mv ~/.local/share/nvim ~/.local/share/nvim.bak

# clone (https)
git clone https://github.com/AstroNvim/AstroNvim ~/.config/nvim

# or clone (ssh)
git clone git@github.com:AstroNvim/AstroNvim ~/.config/nvim
```

After cloning you need to create your user configuration. To do that go to [this repo](https://github.com/AstroNvim/user_example)
and press on the `use this template` button. Once you have that repo in your github account, clone it:

```sh
# create your user config by cloning the template

# if you're using https to clone
git clone https://github.com/<user>/<template-repo> ~/.config/nvim/lua/user

# or run this if you have ssh setup
git clone git@github.com:<user>/<template-repo> ~/.config/nvim/lua/user
```

Next time you open neovim the plugins will be installed:

```sh
nvim
```

### Setting up Copilot

Go to the user config folder you cloned above and edit the `plugins/user.lua` file by adding the
following plugin:

```sh
nvim ~/.config/nvim/lua/user/plugins/user.lua
```

```lua
return {
  -- {
  --   existing plugin
  -- },
  {
    "github/copilot.vim",
    event = "InsertEnter",
    lazy = false,
    autoStart = true,
    config = function()
      vim.g.copilot_assume_mapped = true
      vim.api.nvim_set_keymap("i", "<C-L>", 'copilot#Accept("<CR>")', { silent = true, expr = true })
      vim.g.copilot_no_tab_map = true
    end,
  },
}
```

The lua code above will add copilot to neovim and it will available on insert mode. In my case
I have the `tab` key, which is the default key to accept copilot suggestions, already mapped to something else.
To accept suggestions I'm using `CTRL+L` but you could use some other shortcut. Now save the file and close neovim.
Next time you open it the copilot plugin will be installed and once it is installed type in the following command
and hit enter:

```
:Copilot setup
```

Running it will show a nvim notification with with a code like this `XXXX-XXXX` as a notifiaction. Copy it and then hit enter again and
it will open your browser (you should be logged in your Github account) on a Github tab where you need to paste the code
to authorize the Copilot plugin. That is it. When you reopen neovim the plugin should be ready to give you suggestions while
you're coding.

### Copilot examples

Here are some examples while working on a `contact.$contactId.tsx` route in Remix.

Given the variable name Copilot gives me the result in milliseconds which is what I
expected.
![Copilot suggests result in millis](/img/copilot-suggest-result-in-millis.png)

On this example below, the moment I typed `export const act` Copilot gave the function signature, which I accepted
and then once within the function body without typing anything Copilot suggested the function
implementation which was in most part correct and that saved me a lot of typing.
![Copilot suggests function code](/img/copilot-suggest-action-function-code.png)

Thanks for taking the time to read this article and I hope it has been useful to you.

[1]: https://docs.github.com/en/copilot/quickstart#introduction
