# How to Use Vim

## Notes on Vim

Vim is a "Modal" Text Editor, meaning it has modes, such as **Command**, **Normal**, and **Insert** mode. Vim starts in **Normal** mode when first entering the program.

## Basic Vim Usage

### Changing Modes

* In **Normal** mode press `i` to enter **Insert** mode to edit the file.
* In **Insert** mode press `Esc` to enter **Normal**.
* In **Normal** mode press `:` to enter **Command** mode to enter commands.

#### While in Normal mode:

* To show line numbers, enter `set number`.
* To delete a single line, enter `d` followed by another `d`.
  * Note: No characters will show up on the screen when doing this.
  * Note: Adding a number, such as `4` before the `dd` will cause that number of lines to be deleted.
* To undo previous actions, enter `u`
  * Note: This can be done repeatedly to undo the last several actions.
* To redo actions after undoing them, enter `Ctrl`+ `r`.
  * Note: Like undoing, redoing can be done repeatedly.
* To search for a string of characters, enter `/` followed by your string of interest, such as `Hello`.
  * **Note: For more on Vim Search functions, see [Link].**
  * Note: Pressing `Enter` or `Return` will take your cursor to the first letter of the first match to your search.
  * Note: Pressing `n` will take you to the next match to your search.
  * Note: Pressing `Shift` + `n` will take you to the previous match to your search.

#### While in Command mode:

* To exit **without saving** Vim, enter `:q`.
* To save and exit Vim, enter `:wq`.
* To search and replace a string, enter `:%s/string-to-find/string-to-replace-it-with/gc`
  * Note: The `g` option replaces all search results.
  * Note: The 'c' option requests confirmation of each replacment.

#### While in Insert mode:

### Motion Basics: Traversing the file (Normal Mode)

* `h`, `j`, `k`, and `l` will move your cursor left a character, down a line, up a line, and right a character respectivly.
  * Note: Adding a number infront of the letter, such as `4`, will move the cursor that number of characters or lines.
  * Note: Instead of hitting down arrow 10 times, one could type `10j`.
* To move forward to the beginning of the next word, use `w`.
  * Note: This moves the cursor to the next non-character character, anything not in a-z, A-Z, and 0-9.
* To move backward to the beginning on the previous word, use `b`.
  * Note: This moves the cursor back to the next non-character character, anything not in a-z, A-Z, and 0-9.
* To move forward to the beginning of the next **WORD**, use `shift` + `w`.
  * Note: This moves the cursor to the next non-white-space character.
* To move backward to the beginning on the previous **WORD**, use `shift` + `b`.
  * Note: This moves the cursor back to the next non-white-space character.
* To move forward to the next end of word, use `e`.
  * Note: This will take you to the end of the current word.
  * Note: Word in this instance means the next non-character, anything not in a-z, A-Z, and 0-9.
To move forward to the next end of **WORD**, use `shift` + `e`.
  * Note: This will take you to the end of the current **WORD**.
  * Note: Word in this instance means the next non-white-space character.

### Intermediate Motions: Fine traversal of the file (Normal Mode)

* The sequence `[n]f<x>` will move the cursor forward to the [nth] <x>.
  * Note: The cursor will be on the `x`.
* The sequence `[n]F<x>` will move the cursor backward to the [nth] <x>.
  * Note: The cursor will be on the `x`.
* The sequence `[n]t<x>` will move the cursor forward to the [nth] <x>.
  * Note: The cursor will be on the character before the `x`.
* The sequence `[n]T<x>` will move the cursor backward to the [nth] <x>.
  * Note: The cursor will be on the character before the `x`.

### Intermediate Searching Functions

* To search forward from the top of the file for a string, enter `/` followed by your string.
  * Note: Pressing `n` will take you to the next match to your search.
  * Note: Pressing `Shift` + `n` will take you to the previous match to your search.
  * Note: Regular expressions are permitted in search function.
* To search backward for a string, enter `?` followed by your string.
  * Note: This is not recommended due to the flipping of the `n` and `Shift` + `n` functions.
* To search forward for the word currently under the cursor, enter `*`.
  * Note: This is a bounded search, meaning only exact matches for the search string will show up. `For` will not find `Forward` for example.
  * Note: Use `g*` for an unbounded search. This will find `Forward` when searching for `For`.
 * To search backward for the word currently under the cursor, enter `#`.
   * Note: This is a bounded search, meaning only exact matches for the search string will show up. `For` will not find `Forward` for example.
    * Note: Use `g#` for an unbounded search. This will find `Forward` when searching for `For`.
 
