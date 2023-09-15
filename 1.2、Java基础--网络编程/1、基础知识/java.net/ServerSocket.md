
# 主要方法

## accept() : Socket

```java
public Socket accept() throws IOException {  
    if (isClosed())  
        throw new SocketException("Socket is closed");  
    if (!isBound())  
        throw new SocketException("Socket is not bound yet");  
    Socket s = new Socket((SocketImpl) null);  
    implAccept(s);
    return s;  
}
```

**相关方法 :**

**implAccept(Socket s) : void**

```java
protected final void implAccept(Socket s) throws IOException {  
    SocketImpl si = s.impl;  
  
    // Socket has no SocketImpl  
    if (si == null) {  
        si = implAccept();  
        s.setImpl(si);  
        s.postAccept();  
        return;
    }
  
    // Socket has a SOCKS or HTTP SocketImpl, need delegate  
    if (si instanceof DelegatingSocketImpl) {  
        si = ((DelegatingSocketImpl) si).delegate();  
        assert si instanceof PlatformSocketImpl;  
    }  
  
    // Accept connection with a platform or custom SocketImpl.  
    // For the platform SocketImpl case:    // - the connection is accepted with a new SocketImpl    // - the SO_TIMEOUT socket option is copied to the new SocketImpl    // - the Socket is connected to the new SocketImpl    // - the existing/old SocketImpl is closed    // For the custom SocketImpl case, the connection is accepted with the    // existing custom SocketImpl.    ensureCompatible(si);  
    if (impl instanceof PlatformSocketImpl) {  
        SocketImpl psi = platformImplAccept();  
        si.copyOptionsTo(psi);  
        s.setImpl(psi);  
        si.closeQuietly();  
    } else {  
        s.impl = null; // temporarily break connection to impl  
        try {  
            customImplAccept(si);  
        } finally {  
            s.impl = si;  // restore connection to impl  
        }  
    }  
    s.postAccept();  
}
```

**相关方法**

