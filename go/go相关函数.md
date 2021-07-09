# go相关函数



| 函数名称                        | 功能                                                         | 用例                                                        | 对应Java函数    |
| ------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- | --------------- |
| `reflect.DeepEqual()`           | 比较对象内容                                                 | reflect.DeepEqual(p1, p2)                                   | String.equals() |
| sync.RWMutex                    | 读写锁                                                       | mapLock.Lock()<br />mapLock.Unlock()                        |                 |
| net.Listen()                    | Listen函数创建服务器                                         | net.Listen("tcp", fmt.Sprintf("%s:%d", this.Ip, this.Port)) |                 |
| io.Copy(os.Stdout, client.conn) | //一旦client.conn有数据，就直接copy到stdout标准输出上, 永久阻塞监听 |                                                             |                 |
| time.After()                    | time.After(time.Second * 1)                                  | 定时器,1秒后启动                                            | sleep（1）      |
| fmt.Scanln()                    | fmt.Scanln(&chatMsg)                                         | 输入值                                                      |                 |

