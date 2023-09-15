
# 从xml中加载Properties

xml文件需要有以下声明。
```xml
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
```
下面是源码。
```java
public synchronized void loadFromXML(InputStream in) throws IOException, InvalidPropertiesFormatException{ 
	...
    PropertiesDefaultHandler handler = new PropertiesDefaultHandler();  
    handler.load(this, in);  
    in.close();  
}
```
# 从InputStream中加载Properties
```java
public synchronized void load(InputStream inStream) throws IOException {  
	... 
    load0(new LineReader(inStream));  
}
private void load0(LineReader lr) throws IOException {  
    StringBuilder outBuffer = new StringBuilder();  
    int limit;  
    int keyLen;  
    int valueStart;  
    boolean hasSep;  
    boolean precedingBackslash;  
  
    while ((limit = lr.readLine()) >= 0) {  
        keyLen = 0;  
        valueStart = limit;  
        hasSep = false;  
  
        //System.out.println("line=<" + new String(lineBuf, 0, limit) + ">");  
        precedingBackslash = false;  
        while (keyLen < limit) {  
            char c = lr.lineBuf[keyLen];  
            //need check if escaped.  
            if ((c == '=' ||  c == ':') && !precedingBackslash) {  
                valueStart = keyLen + 1;  
                hasSep = true;  
                break;            } else if ((c == ' ' || c == '\t' ||  c == '\f') && !precedingBackslash) {  
                valueStart = keyLen + 1;  
                break;            }  
            if (c == '\\') {  
                precedingBackslash = !precedingBackslash;  
            } else {  
                precedingBackslash = false;  
            }  
            keyLen++;  
        }  
        while (valueStart < limit) {  
            char c = lr.lineBuf[valueStart];  
            if (c != ' ' && c != '\t' &&  c != '\f') {  
                if (!hasSep && (c == '=' ||  c == ':')) {  
                    hasSep = true;  
                } else {  
                    break;  
                }  
            }  
            valueStart++;  
        }  
        String key = loadConvert(lr.lineBuf, 0, keyLen, outBuffer);  
        String value = loadConvert(lr.lineBuf, valueStart, limit - valueStart, outBuffer);  
        put(key, value);  
    }  
}
```

# 例子

下面是业务逻辑，使用从xml文件中加载Properties的功能。（借助Spring源码）
```java
// --------------------------- PropertiesLoaderUtils.loadProperties() --------------------------------
// Resource可以是 UrlResource 
public static Properties loadProperties(Resource resource) throws IOException {  
    Properties props = new Properties();  
    fillProperties(props, resource);  
    return props;  
}
// --------------------------- PropertiesLoaderUtils.fillProperties() --------------------------------
public static void fillProperties(Properties props, Resource resource) throws IOException {  
    try (InputStream is = resource.getInputStream()) {  
		String filename = resource.getFilename();  
        if (filename != null && filename.endsWith(XML_FILE_EXTENSION)) {  
            if (shouldIgnoreXml) {  
	            throw new UnsupportedOperationException("XML support disabled");  
	        }  
            props.loadFromXML(is);  
        }  
        else {  
            props.load(is);  
        }  
    }  
}
```
