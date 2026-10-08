- Nginx startup failed, prompt that port 80 is occupied
```
nginx: [emerg] bind() to 0.0.0.0:8080 failed (48: Address already in use) 
nginx: [emerg] bind() to 0.0.0.0:8080 failed (48: Address already in use) 
nginx: [emerg] bind() to 0.0.0.0:8080 failed (48: Address already in use) 
nginx: [emerg] bind() to 0.0.0.0:8080 failed (48: Address already in use) 
nginx: [emerg] bind() to 0.0.0.0:8080 failed (48: Address already in use) 
nginx: [emerg] still could not bind()
```
> I tried multiple methods, including finding a process, killing a process, and using the network monitor to find a process, but none of them were successful.
- Final solution
```
lsof -iTCP -sTCP:LISTEN -P | grep :80
```
> Find which process is listening on port 80

> If there is no lsof, install lsof first
```
brew install lsof
``` 
- Finally, kill the process
```
sudo kill  PID 
```