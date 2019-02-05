## How to Use Vim

### Notes on Vim

Vim is a "Modal" Text Editor, meaning it has modes, such as **Command**, **Normal**, and **Insert** mode. Vim starts in **Normal** mode when first entering the program.

### Basic Vim Usage

#### Changing Modes

* In **Normal** mode press `i` to enter **Insert** mode to edit the file.
* In **Insert** mode press `Esc` to enter **Normal**.
* In **Normal** mode press `:` to enter **Command** mode to enter commands.

#### While in Normal mode:

* To show line numbers, enter `set number`.
* To delete a single line, enter `d` followed by another `d`.
..* Note: No characters will show up on the screen when doing this.
..* Note: Adding a number, such as `4` before the `dd` will cause that number of lines to be deleted.
* To undo previous actions, enter `u`
..* Note: This can be done repeatedly to undo the last several actions.
* To redo actions after undoing them, enter `Ctrl`+ `r`.
..* Note: Like undoing, redoing can be done repeatedly.
* To search for a string of characters, enter `/` followed by your string of interest, such as `Hello`.
..* Note: Pressing `Enter` or `Return` will take your cursor to the first letter of the first match to your search.
..* Note: Pressing `n` will take you to the next match to your search.
..* Note; Pressing `Shift` + `n` will take you to the previous match to your search.

#### While in Command mode:

* To exit **without saving** Vim, enter `:q`.
* To save and exit Vim, enter `:wq`.
* To search and replace a string, enter `:%s/string-to-find/string-to-replace-it-with/gc`
..* Note: The `g` option replaces all search results.
..* Note: The 'c' option requests confirmation of each replacment.

#### While in Insert mode:
