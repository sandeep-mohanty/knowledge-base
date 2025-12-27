# A beginner's introduction to Bash parameter expansions

Have you ever stared at a large directory of file names and dreaded the idea of renaming them all? Or perhaps your scripts are littered with [conditional statements](https://www.howtogeek.com/devops/conditional-testing-in-bash-if-then-else-elif/) checking to see if an [environment variable](https://www.howtogeek.com/668503/how-to-set-environment-variables-in-bash-on-linux/) exists. Bash provides a concise syntax for [changing strings](https://www.howtogeek.com/812494/bash-string-manipulation/) that makes life easier. They're called parameter expansions, and I'll show you how to use them.

At a basic level, parameter expansion means changing Bash syntax into a value—expanding it. For example:

```
echo "$foo"
```

This simple [variable](https://www.howtogeek.com/442332/how-to-work-with-variables-in-bash/) turns into its assigned value. However, it's the least useful form of parameter expansion, and there are dozens of others that provide more utility. I'll cover the most important ones and present real-world scenarios where they're effective. At your convenience, you can use them in scripts or on the [command line](https://www.howtogeek.com/terminal-vs-command-line-vs-shell-vs-console/#what-is-a-command-line).


## Changing case

You can change the case of an entire string, or part of it, using only Bash.

### The basics

Change an entire string into uppercase using "^^":

```
name="Hello, World!"
echo "${name^^}"
```

   ![A terminal window displays the text Hello World in uppercase.-2](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-hello-world-in-uppercase-2.png?q=70&fit=crop&w=825&dpr=1)

Change the first character into uppercase using "^":

```
name="hello, World!"
echo "${name^}"
```

   ![A terminal window displays the text Hello World with the first letter uppercase.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-hello-world-with-the-first-letter-uppercase.png?q=70&fit=crop&w=825&dpr=1)

Change the entire string to lowercase using ",,":

```
name="HELLO, WORLD!"
echo "${name,,}"
```

   ![A terminal window displays the text hello world in lowercase.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-hello-world-in-lowercase.png?q=70&fit=crop&w=825&dpr=1)

Change the first letter into lowercase using ",":

```
name="Hello, world!"
echo "${name,}"
```

   ![A terminal window displays the text hello world with the first letter lowercase.-1](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-hello-world-with-the-first-letter-lowercase-1.png?q=70&fit=crop&w=825&dpr=1)

### Real-world examples

How do we use these in practice?

In one scenario, you may find many files with undesirable uppercase file names (e.g., IMG\_2025-01-01.jpg), and you want to [batch rename](https://www.howtogeek.com/different-ways-to-batch-rename-files-in-linux/) them to lowercase:

```
for file in IMG_*.jpg; do
    mv "$file" "${file,,}"
done
```

That will also lowercase file extensions (covered later) and paths (if included).

Now imagine a text document whose items should be uppercase. For example, a list of [MAC addresses](https://www.howtogeek.com/764868/what-is-a-mac-address-and-how-does-it-work/):

```
while read mac; do
    echo "${mac^^}"
done < mac_addresses.txt > mac_addresses_upper.txt
```

I sometimes create [prompts](https://www.howtogeek.com/808593/bash-script-examples/#handling-user-input) for my scripts to evaluate yes or no answers. It's better to convert the string before comparison instead of writing multiple clauses:

```
read -p "Continue? (Y/n): " answer
if [[ "${answer^^}" == "N" ]]; then
    echo "Stopping..."
    exit 1
fi
# This is yes!
```

## Using default values

Encountering unset variables is common when scripting. Often your script needs to ask questions about values before using them. That can clutter your code with unsightly conditional statements. Luckily, Bash has a concise way to manage these scenarios.

### The basics

If a variable is unset or empty, you can fall back to a default value with ":-":

```
echo "${NAME:-value}"
```

   ![A terminal window displays the text value.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-value.png?q=70&fit=crop&w=825&dpr=1)

With a simple "-" (no colon), you can fall back to a default value if the variable was never declared (unset):

```
echo "${NAME-value}"
```

   ![A terminal window displays the text value.-1](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-value-1.png?q=70&fit=crop&w=825&dpr=1)

The ":+" operator is the opposite of ":-": Fall back if "NAME" is set and not empty:

```
export NAME="foo"
echo "${NAME:+value}"
```

   ![A terminal window displays the text value.-2](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-value-2.png?q=70&fit=crop&w=825&dpr=1)

For "+," "NAME" must be set or empty before it falls back to the default value:

```
export NAME="foo"
echo "${NAME+value}"
```

   ![A terminal window displays the text value.-3](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-value-3.png?q=70&fit=crop&w=825&dpr=1)

The "=" operator is a little different, because it does two things: sets and returns the value. For ":=", Bash does this if "NAME" is unset or empty:

```
                       # "NAME" is unset at this point.
echo "${NAME:=value}"  # Expands to "value", but also sets "NAME" to "value".
echo "$NAME"           # "NAME" is now set to "value"
```

   ![A terminal window displays the text value twice.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-value-twice.png?q=70&fit=crop&w=825&dpr=1)

When we exclude the colon ("="), it means set and return the value when "NAME" is unset:

```
echo "${NAME=value}"
echo "$NAME"
```

   ![A terminal window displays the text value twice.-1](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-value-twice-1.png?q=70&fit=crop&w=825&dpr=1)

Since these rules are confusing, here's a summary:

```
# Fallback to the default if "NAME"...
echo "${NAME:-value}"  # Is unset or empty.
echo "${NAME-value}"   # Is unset.
echo "${NAME:+value}"  # Is set and not empty.
echo "${NAME+value}"   # Is set or empty.
echo "${NAME:=value}"  # Is unset or empty.   (Also, assign to "NAME.")
echo "${NAME=value}"   # Is unset.            (Also, assign to "NAME.")
```

### Real-world examples

The sensible way to locate common home directories is to use the constants provided by the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir/latest/) (aka XDG BDS), which is provided by freedesktop.org. However, they're not always set, so we should fall back to a sensible default:

```
mv foo.conf "${XDG_CONFIG_HOME:-$HOME/.config}"
```

Perhaps your script has subsequent file operations, and you don't want to repeat yourself:

```
mv foo.conf "${XDG_CONFIG_HOME:=$HOME/.config}" # Also sets a value.
mv bar.conf "$XDG_CONFIG_HOME" # We don't need to check.
```

If not already set, that will assign the appropriate XDG value on first use.

You can even use parameter expansion as a cheap, inline conditional:

```
export DEBUG=true  # MUST "export" if executing in the CLI.
echo "Running script${DEBUG:+ (debugging enabled)}..."  # Extra message, if debugging.
```

This will print a special message only when debugging is active.

## Changing parts of a string

Replacing part of a string is common on the command line, and not just in scripts. For example, bulk changing file names or manipulating user input. Often we'd use sed, but bash has built-in support for [pattern matching](https://www.howtogeek.com/i-found-using-grep-or-sed-in-bash-scripts-is-painfully-slow-but-heres-how-i-fixed-it/) that's more convenient and sometimes faster.

### The basics

Replacing the first occurrence of a string is straightforward using "/":

```
text="Hello, World!"
echo "${text/World/Universe}"
```

   ![A terminal window displays the text Hello, Universe!](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-hello-universe.png?q=70&fit=crop&w=825&dpr=1)

But you can replace _all_ occurrences using "//":

```
text="Hello, World! Hello, Universe!"
echo "${text//Hello/Goodbye}"
```

   ![A terminal window displays the text Goodbye, World! Goodbye, Universe!](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-goodbye-world-goodbye-universe.png?q=70&fit=crop&w=825&dpr=1)

You can even replace the start of a string (if it matches) with "/#":

```
text="Hello, World!"
echo "${text/#Hello/Goodbye}"
```

   ![A terminal window displays the text Goodbye, World!](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-goodbye-world.png?q=70&fit=crop&w=825&dpr=1)

Or you can replace the end of a matching string using "/%":

```
text="Hello, World!"
echo "${text/%World\!/Universe\!}"
```

   ![A terminal window displays the text Hello, Universe!-1](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-hello-universe-1.png?q=70&fit=crop&w=825&dpr=1)

Be sure to escape problematic special characters ("!"), because they sometimes have special meaning and confuse the shell.

You can also remove parts of strings. The "#" will remove the shortest matching instance:

```
file="foofoo.txt"
echo "${file#foo}"
```

   ![A terminal window displays the text foo.txt.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-foo-txt.png?q=70&fit=crop&w=825&dpr=1)

Let's re-execute the previous command while including a [wildcard](https://www.howtogeek.com/how-to-use-wildcards-in-the-linux-terminal-to-list-files/) to match as much of the string as possible:

```
file="foofoo.txt"
echo "${file#foo*}"
```

   ![A terminal window displays the text foo.txt.-1](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-foo-txt-1.png?q=70&fit=crop&w=825&dpr=1)

Notice the results are the same? That's because "#" is lazy and will match as little as possible. It matched "foo," then gave up. The wildcard had no effect.

In contrast, using "##" removes the longest possible match (aka greedy):

```
file="foofoo.txt"
echo "${file##foo*}"
```

That pattern matched the entire string, so there's no output.

The "%" and "%%" operators work the same as the "#" and "##," except in reverse. Instead of working left to right, they move right to left.

The "%" operator removes the shortest possible match, from right to left:

```
file="foo.bar.txt"
echo "${file%.*}"
```

   ![A terminal window displays the text foo.bar.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-foo-bar.png?q=70&fit=crop&w=825&dpr=1)

The ".\*" pattern will match a period and anything after it. In the string, notice there are two periods (three segments), but only the rightmost segment was matched?

A "%%" removes the longest possible match, from right to left:

```
file="foo.bar.txt"
echo "${file%%.*}"
```

   ![A terminal window displays the text foo.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-foo.png?q=70&fit=crop&w=825&dpr=1)

That's the same pattern, except it uses "%%." It moved right to left, matching ".\*" as it went. The leftmost segment didn't match because it doesn't begin with a period.

This was also a complicated section, so it deserves a cheat sheet:

```
echo "${var/pattern/value}"     # "/"   Replace the first occurrence.
echo "${var//pattern/value}"    # "//"  Replace all occurrences.
echo "${var/#pattern/value}"    # "/#"  Replace at the start.
echo "${var/%pattern/value}"    # "/%"  Replace at the end.
echo "${var#pattern}"           # "#"   Remove the shortest match (left to right).
echo "${var##pattern}"          # "##"  Remove the longest match (left to right).
echo "${var%pattern}"           # "%"   Remove the shortest match (right to left).
echo "${var%%pattern}"          # "%%"  Remove the longest match (right to left).
```

 ![Konsole Terminal open on the Kubuntu Focus Ir14 Linux laptop.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2024/08/52972123178_c7bc15383d_o.jpg?q=49&fit=crop&w=220&h=182&dpr=2)

Related

### Real-world examples

Imagine you have a list of files, and you want to change their extension:

```
for f in *.txt; do
  mv "$f" "${f/%.txt/.md}"
done
```

"%" replaces the end of a string.

Your digital camera creates images with an annoying prefix, so you want to remove them:

```
for f in IMG_*.jpg; do
    mv "$f" "${f#IMG_}"
done
```

"#" removes the start of a string.

Perhaps you've got a path to a file, and you want only the file name:

```
path="/home/user/documents/report.pdf"
echo "${path##*/}"
```

   ![A terminal window displays the text report.pdf.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-report-pdf.png?q=70&fit=crop&w=825&dpr=1)

"##" removes the longest match, which enables a greedy wildcard, matching up to the last slash.

Perhaps you want only the file extension:

```
filename="archive.tar.gz"
echo "${filename##*.}"
```

   ![A terminal window displays the text gz.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-gz.png?q=70&fit=crop&w=825&dpr=1)

Lastly, perhaps you want the base name of a file without the extension:

```
filename="document.tar.gz"
echo "${filename%%.*}"
```

   ![A terminal window displays the text document.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/12/a-terminal-window-displays-the-text-document.png?q=70&fit=crop&w=825&dpr=1)

"%%" is greedy, from right to left, stopping at the last segment (left).

 ![Laptop with the Linux terminal open.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2024/05/laptop-with-linux-terminal-open.jpg?q=49&fit=crop&w=220&h=182&dpr=2)

Related

* * *

To me, parameter expansions are no different from a general programming concept called expressions. It simply means turning a piece of code into a value at runtime. It's a valuable concept that can improve your scripts. It can potentially minimize your use of conditional statements or needlessly slow calls to external tools (such as sed).

You can mix and match parameter expansions as you need. For example, you can embed them:

```
echo "${FOO:-${BAR:-bar}}"
```

That will default to "bar" if neither "FOO" nor "BAR" is set. Keep in mind that while conditional statements are ugly, they are sometimes easier to read. So, if you're deeply nesting parameter expansions, rethink your approach and consider "dumber" code—like conditional statements.

 ![Linux mascot using a laptop with some multiplexer terminals around it.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/05/linux-mascot-terminal-multiplexer.png?q=49&fit=crop&w=220&h=182&dpr=2)

Related