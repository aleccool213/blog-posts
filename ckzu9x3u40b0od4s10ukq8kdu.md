## Why I use Fish Shell over Bash and Zsh 🐟

One of the main allurements of Apple is that things "just work". Most of the people who use their products are covered with the features they release and Apple spends little time on anything else. The features they do ship, are **polished**, have **sensible defaults**, and are **intentional**. This is what I believe the [Fish shell](https://fishshell.com) has become. No wasted time scouring the web for config files others have shared, the best plugins to use, or how to get integrations working with your particular setup.

This shell is meant for **most** people, Fish stands for **Friendly Interactive Shell**, that's why I can recommend it to anyone I work with. They have a very detailed [design document](https://fishshell.com/docs/current/design.html). It's not meant for the likes of system admins who are constantly logging into multiple servers a day. It will never be the default installed shell on most operating systems.

Once you install it, `brew install fish`, you are off to the races. You have a shell where you can become super productive and your favourite tools work as intended. It doesn't try to be the best at everything, but nails the essential core features which make the user experience extremely enjoyable.

## TLDR

- Syntax highlighting
- Inline auto-suggestions based on history
- Tab completion using man page data
- Intuitive wildcard support
- Web based configuration

<table class="image">
  <tr><td style="text-align: center;"><img src="https://media.giphy.com/media/QC7Pr3M4gN0yuEDGgj/giphy.gif" alt=""/></td></tr>
</table>

### Let's break it down

#### Syntax highlighting

My worst memories of bash come from the absence of this feature, [syntax highlighting](https://fishshell.com/docs/current/tutorial.html#tut_syntax_highlighting). A simple thing which makes you think, "wow, now I am using a shell from the 90's"! You can notice it working in the below gif when I try to go to `folder_that_doesnt_exist`, the text turns red. The text then turns blue when it's a valid command.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1645301303053/saTOd3_Mj.gif)

#### Inline auto-suggestions based on history

Smart [auto-suggestions](https://fishshell.com/docs/current/index.html#autosuggestions) are seldom seen, let alone built-in. Instead of just beating the competition, the Fish team thought to demolish it. Using the history of your commands, it suggests commands which you can complete with the `right-arrow key`. You can also, as I do this gif, auto-complete one word or folder at a time with `option + right-arrow key`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1645301304885/Uj7mlY9A2.gif)

> Fun fact, if search results are huge, Fish shell will paginate!

#### Tab completion using man page data

This is because [Fish knows how to parse CLI tool man pages](https://fishshell.com/docs/current/index.html#completion) in many different formats. Git, Docker CLI, package.json, you name it, most commands you try, it will have auto-completions for it.

You can use `tab` to get all the options.

<table class="image">
  <caption align="bottom">All npm scripts for this package, with values of what they actually run, IN THE TERMINAL WUT
</caption>
  <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1565390833/fish-post/2019-08-09_18.46.53.gif" alt="example-of-fish-shell-tab-complete"/></td></tr>
</table>

#### Intuitive wildcard support

In bash, I never liked having to use different flags for selecting files or contents of a folder.

Regularly, this would be done with:

```bash
rm -r folder_1
```

I have always been a fan of familiarity, and [wildcards](https://fishshell.com/docs/current/tutorial.html#tut_wildcards) are just that. You can use them in any command filter down the exact files you need with ease.

e.g.

```
ls *.jpg
```

<table class="image">
  <caption align="bottom">How I feel while using Fish
</caption>
  <tr><td style="text-align: center;"><img src="https://media.giphy.com/media/26tPplGWjN0xLybiU/giphy.gif" alt=""/></td></tr>
</table>

#### Web based configuration

Type in:

```
web_config
```

and you get an entire website dedicated to messing around with any config you do need to touch.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1645301306692/Z8d4l2ENz.png)

### A tiny customization needed to go the extra mile

There aren't a lot of extra packages needed for Fish. Personally, I only use 2, which is wild because at one point I know my Oh-My-Zsh plugins were past 10.

#### Oh My Fish

A homage to the great Oh My Zsh, `omf` is the most popular package manager for Fish. I use this to install just two packages, one for [nvm](https://github.com/derekstavis/plugin-nvm) and one for [spacefish](https://github.com/matchai/spacefish/).

#### SpaceFish

Special mention to [Spacefish](https://github.com/matchai/spacefish/) for being the best shell prompt I have ever used. Support for showing:

- Current Git branch and rich repo status
- Current Node.js version, through nvm
- Package version, if there is a package in the current directory (package.json for example)

<table class="image">
  <caption align="bottom">Spacefish example
</caption>
  <tr><td><img src="https://res.cloudinary.com/dscgr6mcw/image/upload/v1565391692/fish-post/spacefish_example.png" alt="spacefish-shell-prompt-example"/></td></tr>
</table>

#### Config file

You also have access to a config file at `.config/fish/config.sh`. This is where you can set aliases up or set some extra path extensions.

### Caveats

Not being POSIX compliant can scare some developers away. But really in my three years of usage (mostly Node.js, javascript, ruby, e.t.c.), I have not encountered any issues. Some commands I get from the internet which are Bash specific, I'll just `exit` and then come back to Fish when I finish. [This Stackoverflow post](https://stackoverflow.com/questions/48732986/why-how-fish-does-not-support-posix) goes into it more if you are so inclined.

#### But it's easy to be compatible...

Say you have a bash script to run, with Fish you still can:

```bash
bash script.sh
```

Another tip is that you can put this at the top of the file:

```bash
#!/usr/bin/env bash
```

and then make sure its an executable:

```bash
chmod +x script.sh
```

and voila, you can run it as a regular script:

```bash
./script.sh
```

## Resources:

- [Fish Shell Website](https://fishshell.com/)
- [Fish Shell Syntax Highlighting](https://fishshell.com/docs/current/tutorial.html#tut_syntax_highlighting)
- [Fish Shell Autosuggestions](https://fishshell.com/docs/current/index.html#autosuggestions)
- [Try out the Fish Shell tutorial online](https://rootnroll.com/d/fish-shell/)
- [Oh My Fish Package Manager](https://github.com/oh-my-fish/oh-my-fish)
- [NVM wrapper plugin](https://github.com/derekstavis/plugin-nvm)
- [Spacefish Fish Shell Theme](https://github.com/matchai/spacefish/)
- [List of awesome Fish related software](https://github.com/jorgebucaran/awesome-fish)
- [Fisher, another package manager with a file-based extension config](https://github.com/jorgebucaran/fisher)
  - [A friends fishfile for fisher](https://github.com/elliottsj/dotfiles/blob/master/common/.config/fish/fishfilehttps://github.com/elliottsj/dotfiles/blob/master/common/.config/fish/fishfile)
- [Support Bash scripts in Fish](https://github.com/edc/bass)

> Like this post? Consider [buying me a coffee](https://www.buymeacoffee.com/yourboybigal) to support me writing more. 

> Want to receive quarterly emails with new posts? [Signup for my newsletter](https://mailchi.mp/f91826b80eb3/alecbrunelleemailsignup) 
