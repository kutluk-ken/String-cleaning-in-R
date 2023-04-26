# Working with Strings in R workshop

###### Department of Statistical Sciences, University of Toronto - Lindsay Katz

### 1. Background and Overview

In programming, a string is any value written in single or double quotes, like "stat 101", 'doss' or "2023". The string data type exists to capture text data, as opposed to numeric data which we are most often used to working with in statistics.

In this workshop, you will learn some helpful functions primarily from the `stringr` package [@stringr], which allow you to explore and modify text data. We will also go over an introduction to regular expressions, which is a super powerful tool that allows us to detect and match patterns in text data in a generalized and efficient manner.

Using a number of examples and exercises, this workshop will focus on:

1.  detecting matches,
2.  extracting matches,
3.  string mutation, and
4.  string-based separation

The first three sections include introductions to `stringr` and regular expressions with examples, and the fourth section will apply some of what we've learned to a more complicated text data example.

### 2.The `stringr` package

The `stringr` package is part of the series of packages within the `tidyverse` [@tidyverse]. You can load it in on its own, or by loading the `tidyverse` package:

#### Detecting matches
- `str_detect` - detect matches
- `str_count` - count the number of matches in a string

#### Extracting matches
- `str_extract` and `str_extract_all` - extract matches
- `str_sub` - extract sub-strings from a character vector

#### Mutating strings
- `str_replace` and `str_replace_all` - replace elements of a string
- `str_to_upper`, `str_to_lower`, and `str_to_title` - convert the case of strings
- `str_squish` - trim outer whitespace and clean internal whitespace
- `str_split` - split strings into sub-strings

### 3. Regular Expressions
Regular expressions (regex) are a concise language designed to allow us to search for patterns within strings. These are sequences of characters used to search text for precise matches. Creating a regular expression allows us to check that the string conforms to the specific pattern we have designed.

For example, if you are subscribing to a newsletter, the website will prompt you to enter a valid email address. If your input doesn't match the expected regular expression pattern associated with an email address (e.g. have an '\@' symbol and end with '.com', '.ca', '.org', etc.), that website will throw an error.

When you use functions from the `stringr` package and provide a pattern argument as we have done in the previous examples, this pattern is itself a regular expression. Thus, understanding the proper syntax for regular expressions is crucial when it comes to more complicated pattern matching.

- 3.1 Regex fundamentals

Below is a list of some common character classes (those in square brackets) and match characters. The backslashes are written to denote special characters, because as you'll see later on, many commonly used punctuation characters like periods, parentheses and question marks have special meanings in the context of regular expressions. The technical term for this backslash use is to "escape" special characters.

-   `[:alpha:]` - letters
-   `[:lower:]` - lowercase letters
-   `[:upper:]` - uppercase letters
-   `[:alnum:]` - numbers and letters
-   `[:space:]` - space characters
-   `[:blank:]` - space and tab, but not a newline
-   `[:digit:]` - digits
-   `[:punct:]` - punctuation
    -   This includes: periods, commas, colons, slashes, exclamation and question marks, parentheses, quotation marks and more
-   `\\.` - period
-   `\\!` - exclamation mark
-   `\\?` - question mark -`\\\\` - backslash
-   `\\(` and `\\)` - parentheses
-   `\\n` - newline
-   `\\d` - digit
-   `\\s` - space
-   `\\t` - tab
-   `.` - matches every character except a newline
    -   this special meaning of a period is a prime example of why we need to escape it using backslashes when we want to match for one in a string

Note that in base R, many functions require classes to be written with two sets of square brackets like "`[[:alpha:]]`" [@stringr].

- 3.2 Anchors

By default, regular expression patterns will detect a match in any part of the string. In some cases, you may wish to "anchor" the regular expression to specify the match to be detected from the start or end of the string.

To anchor the regular expression so it searches for a match at the start of the string, use a `^` character before the pattern to be matched. To anchor to match the end of the string, use a `$` character at the end of the pattern to be matched. You can use both anchors if you want to force a regular expression to match only a complete string which matches the specified pattern.

For example, consider the string "doss". If we want to extract the last letter of that string ("s"), we can use the regular expression `[:alpha:]$` to do so. If we just use the regex `[:alpha:]` without the anchor, the first letter match in the string will be returned which in our case is the letter "d".

- 3.3 Quantifiers

Sometimes we want to match for a repeated character or character class in a string, such as three consecutive digits. Adding quantifiers to regular expressions allows us to do so. Here are a few examples of quantifiers that we will use later on, however there are others you can read about [here](https://www.regular-expressions.info/refrepeat.html).

1.  `x{m}` - exactly m
2.  `x{m,n}` - between m and n
3.  `x{m,}` - at least m (i.e. m or more)
4.  `x+` - one or more

For instance, if we are given a string containing a phone number "416-978-2190" and want to match the set of four digits in a row, we would use the regular expression "`\\d{4}`", or equivalently, "`[:digit:]{4}`".

- 3.4 Lookarounds

The last component of regular expressions we will cover is lookarounds. These allow you to improve the precision of matches by specifying what should or should not come before or after a pattern. There are two types of lookarounds: lookaheads (what follows a pattern) and lookbehinds (what precedes a pattern). Lookarounds can be positive or negative.

The table below summarizes each type of lookaround, and the corresponding regex syntax [@stringr].

```{r, echo=FALSE}
tibble("type" = c("positive lookbehind",
                  "negative lookbehind",
                  "positive lookahead",
                  "negative lookahead"),
       "regex syntax" = c("(?<=y)x",
                   "(?<!y)x",
                   "x(?=y)",
                   "x(?!y)"), 
       "description" = c("pattern x is preceded by pattern y",
                        "pattern x is not preceded by pattern y",
                        "pattern x is followed by pattern y",
                        "pattern x is not followed by pattern y")) %>% 
  knitr::kable() 
```

#### 4 Using regex patterns in `stringr` functions


- 4.1 `str_detect`
Before, we used this function to detect strings that contained the regex pattern "cake". Now, let's only detect those where "cake" is not followed by another letter. To do this, we use a negative lookahead.

- 4.2`str_extract`
Using regex, let's extract the following match: one or more letters (`[:alpha:]+`) which are preceded by a single space (`[:space:]`) that continue to the end of the string. (Note: you could also use a literal space `" "` or `\\s` for this).

### 5. Parliamentary Debates Example
In this section we are going to put what we have learned into practice with a more complicated data example.

The data we will be working with is based on a research project I have been working on with Dr. Rohan Alexander, where we have developed scripts to build a database containing 24 years of proceedings of the Australian Parliamentary debates.

These parliamentary debate records are essentially verbatim transcripts of everything said in parliament that day, making them a really valuable source for text analysis purposes, which is why we have worked to create this accessible database which is ready to be analyzed. The official name for these records is "Hansard".

As you will see in the data, there is one row for each unique debate that took place in parliament that day, with a number of columns including specific details on the Member of Parliament (MP) whose turn it was to speak (i.e. the MP who initiated each debate), such as their political party and full name.

To begin, let's read in the CSV file provided called "`hansard_ex.csv`". This is a subset of 10 speeches that took place in the Australian House of Representatives proceedings on November 30th 2021.

