# This program has been vibecoded

The tools used to vibe code this educational only example are

[opencode](https://opencode.ai/)
[Muse Spark 1.3](https://developer.meta.com/ai/models/muse-spark/)

Here is the prompt I used:

```
Write a python program that will help me find document files (txt, md, pdf, docx and other popular text formats) by name or content. When searching by content, it must work as following (A and B are strings):

| search string | result expected |
| A | anything containing A |
| A B | anything containing A AND B |
| A OR B | anything containing A or B |
| "A A A" | the exact composed string |
| "B B B" "A A A" | must contain the exact composed strings |
| "B B B" OR "A A A" | must contain at least one of the two exact composed strings |

The result must be a client (terminal based) program to be called by the user.

Name the program "searcher" and store it on /home/brod/scripts/opencode/searcher/
```
