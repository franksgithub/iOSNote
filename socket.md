# socket


```
- (void)connectSocket {
    int server_socket = socket(AF_INET, SOCK_STREAM, 0);
    if (server_socket == -1) {
        NSLog(@"创建失败");
    } else {
        struct sockaddr_in server_addr;
        server_addr.sin_len = sizeof(struct sockaddr_in);
        server_addr.sin_family = AF_INET;
        server_addr.sin_port = htons(_port);
        server_addr.sin_addr.s_addr = inet_addr([_ip UTF8String]);
        bzero(&(server_addr.sin_zero), 8);
        int aResult = connect(server_socket, (struct sockaddr*)&server_addr, sizeof(struct sockaddr_in));
        if (aResult == -1) {
            NSLog(@"链接失败");
        } else {
            NSLog(@"链接success");
        }
    }
}
```


