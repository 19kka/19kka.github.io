---
layout:     post
title:      Using protocol buffers in socket(Python)
date:       2015-10-13 10:00:18
summary:    Python socket send data with protobuf buffers
categories: protobuf python
---

  I'm tring to write an application with protocol buffers to send data on socket connecet.I encountered some problems , this is my summary:

###Pre preparation

#####1.download Google protocol buffers.

  look at this [https://developers.google.com/protocol-buffers/](https://developers.google.com/protocol-buffers/)

#####2.Buliding Google protocol buffers(Python). 
  if you see '...no module name google.protobuf..' do this in Terminal: 
    
    (sudo)pip install protobuf


#####3.To understanding [struct](https://docs.python.org/2/library/struct.html)

###Main body

####Client

{% highlight python lineno %}
	import edu_pb2
	.
	.
	.
	line = edu_pb2.Eline()
	line.id = 1000
	line.width = 1
	line.color = -16777216
	protobuf_data = line.SerializeToString()#Serialize
	type = 1001
	packed_len = struct.pack('LL',type,len(protobuf_data))
	call(["ls","-l"])
	# str2 = struct.unpack('LL', packed_len)
	# print str2 #(1000,-16777216)
	sock.sendall(packed_len+protobuf_data)
{% endhighlight %}

  
 ---
  
    struct.pack('LL',type,len(protobuf_data))
   
'LL' means two long types,more types looks [here](https://docs.python.org/2/library/struct.html)

###Server
first in server i def a funcation name socket_read_n
{% highlight python lineno %}
	def socket_read_n(sock, n):
    """ Read exactly n bytes from the socket.
        Raise RuntimeError if the connection closed before
        n bytes were read.
    """
    buf = ''
    while n > 0:
        data = sock.recv(n)
        if data == '':
            raise RuntimeError('unexpected connection close')
        buf += data
        n -= len(data)
    return buf
{% endhighlight %}

then .
{% highlight python lineno %}
len_buf = socket_read_n(sock, 8)
            type = struct.unpack('>LL', len_buf)[0]
            msg_len = struct.unpack('>LL', len_buf)[1]
            print(type)#1001
            msg_buf = socket_read_n(sock, msg_len)
            msg = edu_pb2.ELine()
            msg.ParseFromString(msg_buf)
            print(msg)
{% endhighlight %}

enjoy it!

Reference:  
[http://stackoverflow.com/questions/2038083/how-to-use-python-and-googles-protocol-buffers-to-deserialize-data-sent-over-tc](http://stackoverflow.com/questions/2038083/how-to-use-python-and-googles-protocol-buffers-to-deserialize-data-sent-over-tc)  


