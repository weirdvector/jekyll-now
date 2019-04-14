# Useful Vim Shortcuts
 
I recently have been making more of an effort to use git from the command line. While editing a commit message, I was dropped into the default text editor for git: Vim (cue scary music). Chances are you remember vim from school as the text editor you can't quit. It's a modal text editor, which means there are different 'modes' you can use for editing text and inserting text. Once you get used to basic navigation, it quickly becomes apparent that vim is a very intuitive and powerful tool.

Here are some useful shortcuts I've discovered recently while working with vim.

## Editing text within a code block

Let's say you need to delete a parameter from a method signature:

```
void myFunction(string theValue, int someNumber, bool someFlag) {
    // TODO ...
}
```

If I wanted to get rid of the boolean value in the parameters, I could scroll into the parameters until my cursor is over the 'b' in bool, then push ```2dw``` to delete 2 words, or you could use ```d])```. This command means delete until we see a ```)``` character, so it will leave the rest of the line intact. 

If you needed to change all of the parameters, you could use ```ci)``` or change in parentheses. This will delete all of your parameters, leaving your cursor in between the open and close parentheses. Cool!

Also, this command works with other delimiters too, so you can delete or change between brackets, angle brackets and braces too.

## Editing multiple lines at the same time

I remember being blown away the first time I saw Sublime Text's multiple cursor editing. It's so cool! But it's also a feature that's been in vim for a long time. 

To edit multiple lines at the same time, first enter visual block mode by using ```c-V``` (control+shift+v). Then push ```I``` and start typing. It'll look like you're only typing on one line, until you push ```ESC```. This will addthe text you typed into all the lines you had selected earlier. This is a really quick way to comment out multiple lines of code.

## Finding matching parentheses

This is another quick command. A great feature of advanced IDEs is that you can select one bracket, parenthesis or brace and the matching one will be highlighted (or not, if you're missing it). With vim you can push ```%``` while on a parenthesis to jump to its match.

## Registers

When you delete, yank or change text, it's stored in a register until you use it or overwrite the register. Usually this is stored in the so-called 'unnamed' register (```""```). There are a number of other registers that vim uses.

The numeric registers (```"0``` to ```"9```) are a queue of your changes and deletions. Every time you do another deletion or change, the contents of every register is pushed to the next numbered register. 

You can also use a letter to specify the register you want to store your yank or delete in by using ```"a``` before doing the yank or delete.  If you use a lowercase letter, it will overwrite the register, while an uppercase letter will append the text to the register. 

To do this, just push ```"``` (you'll see it show up at the bottom of your screen), then push either a letter or number, then the command you want to use (```d```, ```x```, ```c```, ```y```, etc). If you lose track of what's where and want to see the contents of your registers, use ```:reg```. 

There are also other registers like ```"_``` which is the 'black hole register' that does not change the value of any other register.

This is a really great feature, and is super useful if like me you've ever wanted to copy and paste multiple things at the same time, without ever having to go back to copy again. For help, see ```:help registers```.

## More coming soon
Vim is a powerful tool and I'm honestly floored at the amount of features that are hidden away in it. As I discover more about it (and the ecosystem surrounding it like ctags, netrw and various plugins) I'll add more posts. Thanks for reading!
