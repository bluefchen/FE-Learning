1. 通过一个http-listener.js的中间件，将所有的请求插入二个header：projectId与是否record
2. 对hsf的服务调用，通过在调用前，获取hsf的参数，并存储
3. 通过使用process.nextTick的方法，最后，把http的所有请求，响应，header，body信息存入http link info中。也会把hsf的服务进行存储。