# TTL Values

| TTL Value | Likely OS / Device Type         | Notes                                 |
| --------- | ------------------------------- | ------------------------------------- |
| 255       | Cisco, BSD, Solaris             | Often seen in routers or switches     |
| 128       | Windows (all versions)          | Most Windows versions default to this |
| 64        | Linux, macOS, Android, iOS      | Common Unix-like systems              |
| 60        | AIX                             | IBM's UNIX OS                         |
| 32        | Older Windows (e.g., Win 95/98) | Rarely seen today                     |
| 30        | Some embedded systems           | Can vary by vendor                    |
| 1         | Misconfigured system or hop     | Unlikely to be a real endpoint        |