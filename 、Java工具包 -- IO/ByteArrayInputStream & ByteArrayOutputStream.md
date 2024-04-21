# 一、ByteArrayInputStream

## 私有属性

```java
public class ByteArrayInputStream extends InputStream {
	protected byte buf[];
	protected int pos;
	protected int count;
	protected int mark = 0;
}
```

## 构造方法

```java
public class ByteArrayInputStream extends InputStream {
	public ByteArrayInputStream(byte buf[]) {  
	    this.buf = buf;  
	    this.pos = 0;  
	    this.count = buf.length;  
	}
	public ByteArrayInputStream(byte buf[], int offset, int length) {  
	    this.buf = buf;  
	    this.pos = offset;  
	    this.count = Math.min(offset + length, buf.length);  
	    this.mark = offset;  
	}
}
```

## 常用方法

```java
public class ByteArrayInputStream extends InputStream {
	
}
```


# 二、ByteArrayOutputStream

## 私有属性

```java
public class ByteArrayOutputStream extends OutputStream {
	protected byte buf[];
	protected int count;
}
```

## 构造方法

```java
public class ByteArrayOutputStream extends OutputStream {
	public ByteArrayOutputStream() {  this(32);  }
	public ByteArrayOutputStream(int size) {  
	    if (size < 0) {  
	        throw new IllegalArgumentException("Negative initial size: "  
	                                            + size);  
	    }  
	    buf = new byte[size];  
	}
}

```

# 三、Experiment

## 1.ByteArrayInputStream

```java
	public static void main(String[] args) {
		String mes = "hello,world" ;
		byte[] b = mes.getBytes() ;
		
		ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream( b ) ;
		int result = -1  ;

		while( ( result = byteArrayInputStream.read() ) != -1){
			System.out.println( (char) result );
		}
		try {
			byteArrayInputStream.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```
## 2. ByteArrayInputStream

```java
	public static void main(String[] args) {
		String mes = "你好,world" ;
		byte[] b = mes.getBytes() ;
		ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream() ;
		try {
			byteArrayOutputStream.write( b );
			FileOutputStream fileOutputStream = new FileOutputStream( new File( "F:/123.txt" ) ) ;
			byteArrayOutputStream.writeTo( fileOutputStream ) ;
			fileOutputStream.flush();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}catch (IOException e) {
			e.printStackTrace();
		}finally{
			try {
				byteArrayOutputStream.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

