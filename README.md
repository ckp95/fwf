# fwf - Filter With Feedback

Interactively create text-filtering commands (`sed`, `jq`, `awk`, `grep`, etc) with immediate feedback.

![sed demo](pics/sed_demo.mp4)

![jq demo](pics/jq_demo.mp4)

This is a tool born of frustration with UNIX command-line text-filtering tools, like `sed`, `awk`, `jq`, and so on. Whenever I needed to use one, I would end up in a frustrating loop of "type command -> press enter -> see error / incorrect output -> repeat". I wanted something with **immediate** feedback so I could iteratively construct the filter string. I also didn't want to deal with the maddening incidental complexity that is shell quote-escaping.

So, I hacked together a pair of zsh scripts that let me type in a filter command and see the results live-update with every keypress. On exit, it edits the current zsh buffer to insert the result into the typed command, so you can immediately run it, or continue building up a longer command with more pipes and filters.

It uses the `fzf` utility to render the preview, since I'm lazy and didn't want to code such a thing myself.

I am very happy with the result. `fwf` makes exploration through the solution-space fast and cheap, resulting in less trawling through StackOverflow. The burden on working memory is reduced by showing the before-text, the after-text, and the filter string all on one screen, and by handling shell-escaping automatically. I used to hate `sed`, `awk`, and `jq`, but combined with `fwf` they are a joy to use.

And it's less than 30 SLOC of zsh.

## Installation:

Clone the repo, make `_fwf_main` and `_fwf_columnate` executable via `chmod u+x`, copy them or symlink them somewhere on your `$PATH`, then add the following snippet to your `.zshrc`:

```zsh
function fwf() {
  local result=$(cat /dev/stdin | _fwf_main "$@")
  local last=$history[$HISTCMD]
  local removed_fwf="$(echo -E $last | sed -E 's/(.*)fwf /\1/g')"
  print -z -r "$removed_fwf $result"
}
```

Yeah it's a useless-use-of-cat; blow me.

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

