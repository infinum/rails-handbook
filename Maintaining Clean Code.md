Once in a while, you'll face the challenge of adding a feature to a legacy project. Please follow these rules:

* Follow the Boy Scout Rule—always leave the code behind in a better state than you found it. If you're working with a legacy codebase, try to leave the code you're touching in a better state by not doing hacks, but rather adapting the code to conform to the new feature requirements. Thinking you'll do the refactoring later is wrong because that never happens.
* There is an exception to the previous rule—don't refactor untested code that works—you'll probably break it. The best way would be to test the code that you're working with, and then refactor it. If it is not possible to write tests for that part of the legacy code you're working with, just use the code as is.
