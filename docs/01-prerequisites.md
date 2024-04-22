# Условия

В нашей лабараторке мы создадим виртуалки

## Virtual or Physical Machines



| Name    | Description            | SRV | 
|---------|------------------------|-----|
| jumpbox | Administration host    | 1   | 
| server  | Kubernetes server      | 1   | 
| wrk-1   | Worker-1               | 1   | 
| wrk-2   | Worker-2               | 1   | 
| wrk-3   | Worker-3               | 1   |
| node-1  | Node-1                 | 1   |
| node-2  | Node-2                 | 1   |
| node-3  | Node-3                 | 1   |

И так моя любимая часть тут будут 100% ошибки,
Подзака решить ее вы сможете данной команды
~~~
uname -mov
~~~

Далее: [Настройка jumpbox](02-jumpbox.md)
