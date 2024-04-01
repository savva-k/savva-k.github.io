---
title: "Programming competition and a small Java task"
date: "2016-04-27"
categories:
  - "Coding"
tags:
  - "Java"
  - "task"
---

Last week we had a programming competition at work which consisted of some small tasks. One of them was to check, if we can make a given string palindrome by adding one letter in any place. Shame on me, I didn't manage with this task on time. However I got the third place in this competition. 🏆😁

So the task:  
Your app should check, if it's possible to make a palindrome by adding a lowercase English letter to a given string in any position. If it is, the application should return an index which points to where to put a character, otherwise the app should return `-1`. The given string contains only lowercase Latin characters.

What is a palindrome? Palindrome is a word or a number, which reads the same backward and forward. For example:

- Was it a car or a cat I saw?
- 1234321
- Step on no pets

See the solution.

```java
package com.imsavva.tests.palindrome;

public class PalindromeApp {

    private char[] alphabet = "abcdefghijklmnopqrstuvwxyz".toCharArray();

    public static void main(String[] args) {
        PalindromeApp app = new PalindromeApp();
        String[] testData = { "aaa", "tacocat", "tacoat", "was i a car or a cat i saw", "aaabas" };

        for (String test : testData) {
            System.out.println(app.isPossibleToPlaceCharToMakePalindrome(test));
        }
    }

    public int isPossibleToPlaceCharToMakePalindrome(String str) {
        str = str.replaceAll("\\s", "");

        for (int i = 0; i < alphabet.length; i++) {
            for (int j = 0; j <= str.length(); j++) {
                String test = insertSymbol(str, alphabet[i], j);

                if (isPalindrome(test)) {
                    System.out.println("found!");
                    return j;
                }
            }
        }

        return -1;
    }

    private String insertSymbol(String source, char character, int offset) {
        return new StringBuilder(source).insert(offset, character).toString();
    }

    public boolean isPalindrome(String str) {
        String reverse = new StringBuilder(str).reverse().toString();
        return str.equals(reverse);
    }
}
```
