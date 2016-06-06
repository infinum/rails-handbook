# Maintaining Clean code

Once in a while you'll face a challenge of adding a feature to a legacy project. Please follow these rules:

* Follow the Boy Scout Rule - always leave the code behind in a better state than you found it. If you're working with a legacy codebase, try to leave the code you're touching in a better state by not doing hacks, but rather adapting the code to conform to new feature requirements. Thinking you'll do the refactoring later is wrong since that never happens.
* The previous rule has an exception - don't refactor untested code that works - you'll probably break it. The best way would be to test code that you're working with and then refactor it. If you don't have the possibility to write tests for that part of the legacy code you're working with, just use the code as is.
