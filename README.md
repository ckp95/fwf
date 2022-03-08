# fwf - Filter With Feedback

Interactively create [text-filtering](https://en.wikipedia.org/wiki/Filter_(software)) commands (`sed`, `jq`, `awk`, `grep`, etc) with immediate feedback.

https://user-images.githubusercontent.com/43073868/153730961-0041d2c8-d074-4fe3-9c59-26409a92b214.mp4

https://user-images.githubusercontent.com/43073868/153730962-527ad148-c4ba-4550-a946-4f6c29c986ac.mp4

This is a tool born of frustration with UNIX command-line text-filtering tools, like `sed`, `awk`, `jq`, and so on. Whenever I needed to use one, I would end up in a frustrating loop of "type command -> press enter -> see error / incorrect output -> repeat". I wanted something with **immediate** feedback so I could iteratively construct the filter string. I also didn't want to deal with the maddening incidental complexity that is shell quote-escaping.

So, I hacked together a pair of zsh scripts that let me type in a filter command and see the results live-update with every keypress. On exit, it edits the current zsh buffer to insert the result into the typed command, so you can immediately run it, or continue building up a longer command with more pipes and filters.

It uses the `fzf` utility to render the preview, since I'm lazy and didn't want to code such a thing myself.

I am very happy with the result. `fwf` makes exploration through the solution-space fast and cheap, resulting in less trawling through StackOverflow. The burden on working memory is reduced by showing the before-text, the after-text, and the filter string all on one screen, and by handling shell-escaping automatically. I used to hate `sed`, `awk`, and `jq`, but combined with `fwf` they are fun to use. I hope you find it useful as well.

## Installation:

### Manual
Install the files `fwf`, `_fwf_main`, and `_fwf_columnate` somewhere on
your fpath (for example, if you have `fpath=(~/.zsh $fpath)`, place them
in `~/.zsh`), then include `autoload -U fwf` in your `.zshrc` or another
startup file.

### Using [zinit](https://github.com/zdharma-continuum/zinit)

Add the following to your zsh configuration:
```
zinit ice as"command"
zinit snippet 'https://raw.githubusercontent.com/ckp95/fwf/6a8cec7a401fdb1928648ac4a68077438392243f/_fwf_main'
export FPATH="$FPATH:$(which _fwf_main)"

zinit ice as"command"
zinit snippet 'https://raw.githubusercontent.com/ckp95/fwf/6a8cec7a401fdb1928648ac4a68077438392243f/fwf'

zinit ice as"command"
zinit snippet 'https://raw.githubusercontent.com/ckp95/fwf/6a8cec7a401fdb1928648ac4a68077438392243f/_fwf_columnate'
```

## Usage:

```zsh
echo 'some string' | fwf [COMMAND]
```

This will launch a `fzf` preview with two panes. The left pane is the original piped text, while the right pane is the transformed text. The transformed text is created by running `[COMMAND] '<what you type>'` on every keystroke. So if you run `echo 'quick brown fox' | fwf sed -E` and then type `s/quick/slow/g`, the right pane will show the result of `echo 'quick brown fox | sed -E 's/quick/slow/g'`. When you press return, your command line will be edited to say `echo 'quick brown fox' | sed -E 's/quick/slow/g'`.

You don't need to type the enclosing single-quotes yourself; this is done for you. If there are quotes in your typed string, they will be properly escaped in the result.

This should work with any text-processing tool that reads from standard input, takes a string argument, and writes to stdout.

e.g.

```zsh
cat my_file.txt | fwf sed -E
cat my_file.txt | fwf awk
cat my_file.json | fwf jq
```

## Caveats

- It only works by piping stuff into it, i.e. `something | fzed awk`, not `fzed something awk`.
- It writes the entire piped input into a temporary file and then reads that file into memory, so you might want to cut down big inputs with `head` first.
- It doesn't preserve colors / syntax highlighting.
- It only works in zsh.
- It writes the amended command on a new line in the terminal, rather than editing the existing line. There is probably a way to make it do that by leveraging zsh's tab-completion functionality, but I couldn't figure it out. If you know how to do it, let me know.

## Dependencies:

- `fzf`
- `zsh`

## Isn't this just like [`up`](https://github.com/akavel/up), but more restricted?

Mostly. `fwf` shows both the before and after of the text transformation, which `up` doesn't do. But `up` does let you chain together an arbitrary pipeline, rather than just explore the results of a single command. `fwf` doesn't do that. I don't want that kind of capability because you're one wrong move away from using a dangerous command like `rm` in a long pipeline and suddenly all is lost.

I just wanted something to help me write a single text filter at a time. Also I don't think `up` edits the command buffer on exit, but I suppose you could write a shell alias to make it do that, so that part is a wash.

That said, you can use `fwf` on dangerous commands, but this should stick out like a sore thumb as dangerous. Maybe I should put in a blacklist of commands so `fwf` just fails instantly when you try to use it on them: `rm` and `dd` would be on it for sure.

