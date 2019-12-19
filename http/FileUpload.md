# http文件上传格式

1. 设置contentType
> Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

这里声明上传头，和分割字符串 **boundary**

2. Body体结构
```http{.line-numbers}

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text" 

title 
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="Test"

tests
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--

```

###注意事项

 > 第一行不能是空行，必须是分割符号，value上必须空一行，分割符号前必须多加一个--，最后结尾必须加 -- 加一个\r\n，内容为文件必须加Content-Type

 代码示例：
 
```CSharp
        ///添加文件头
        private byte[] AddFile(string boundary, string name,string filename,Stream file,string ContentType)
        {
            var head= Encoding.UTF8.GetBytes("\r\n--" + boundary + "\r\nContent-Disposition: form-data; name=\"" + name + "\"; filename=\"" + filename + "\"\r\nContent-Type: " + ContentType + "\r\n\r\n");
            byte[] buffer = new byte[1 << 20];
            MemoryStream ms = new MemoryStream();
            int count = 0;
            ms.Write(head, 0, head.Length);
            while ((count = file.Read(buffer, 0, buffer.Length))>0)
                ms.Write(buffer,0, count);
            ms.Close();
            file.Close();
            return ms.ToArray();
        }
        /*
        添加普通头
        */
        private byte[] AddField(IDictionary<string,string> list,string boundary)
        {
            StringBuilder sb = new StringBuilder();
            foreach(var key in list)
            {
                sb.Append("\r\n--"+boundary+ "\r\nContent-Disposition: form-data; name=\"" + key.Key + "\"\r\n\r\n" + key.Value);
            }
            sb.Remove(0, 2);
            return Encoding.UTF8.GetBytes( sb.ToString());
        }
        /**
            发送文件
        */
        public JObject UploadFile(string url, SortedDictionary<string, string> list,IDataInfo[] files)
        {
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
            request.Method = "POST";
            var boundary = CreateSign();//获取分割符号
            request.ContentType = "multipart/form-data; boundary="+ boundary;
            var body = request.GetRequestStream();//获取request请求
            var buffer = AddField(list,boundary);//构建body 文件参数格式体
            body.Write(buffer, 0, buffer.Length);//写入
            for (int i = 0; i < files.Length; i++)
            {
                var bf = AddFile(boundary, "file", files[i].FileName(), files[i].FileBody(), "application/octet-stream");
                body.Write(bf, 0, bf.Length);
            }
            //body.Write(new byte[] { 13, 10 }, 0, 2);
            var endhead = Encoding.UTF8.GetBytes("--" + boundary + "--\r\n");//构建结尾分割符号
            body.Write(endhead, 0, endhead.Length);
            body.Close();
            body.Flush();
            var response = request.GetResponse();
            var resStream = response.GetResponseStream();
            MemoryStream ms = new MemoryStream();
            byte[] tmp = new byte[1 << 15];
            int readCount;
            while ((readCount = resStream.Read(tmp, 0, tmp.Length)) > 0)
                ms.Write(tmp, 0, readCount);
            resStream.Close();
            response.Close();
            return JObject.Parse(Encoding.UTF8.GetString(ms.ToArray()));
        }

```

构建出的结果：
```

------WebKitFormBoundarySS15W9fUdPbnkVtq
Content-Disposition: form-data; name="serw"

wssd
------WebKitFormBoundarySS15W9fUdPbnkVtq
Content-Disposition: form-data; name="Test"

tests
------WebKitFormBoundarySS15W9fUdPbnkVtq
Content-Disposition: form-data; name="file"; filename="test"
Content-Type: application/octet-stream

#!/usr/bin/python
# -*- coding: UTF-8 -*-


import os;
import urllib.request
import json;



res=os.open('test.txt',os.O_RDWR|os.O_CREAT);
text= os.read(res,10000);

text=str(text);

text= text.replace('component: Layout,','');
text= text.replace('() => import(','');
text= text.replace("'),","',");


os.write(res,text.encode('utf-8'));
os.close(res);






------WebKitFormBoundarySS15W9fUdPbnkVtq--

```