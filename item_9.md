# 아이템9. try-finally보다는 try-with-resources를 사용하라

----
### try-finally 방식 1개면 상관없어보인다.
```java
static String firstLineOfFile(String path) throws IOException {
    BuffreredReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

### try-finally 방식 중첩된경우 복잡해진다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }   
        } finally {
            out.close();
        }   
    } finally {
        in.close();
    }
}
```

### 복수의 자원을 처리하는 try-with-resources 
```java
static void copy2(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src)) {
        try (OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        }
    }
}

static void copy3(String src, String dst) throws IOException {
    try (InputStream in = Files.newInputStream(Paths.get(src));
         OutputStream out = Files.newOutputStream(Paths.get(dst))) {
        byte[] buf = new byte[1024];
        int n;
        while ((n = in.read(buf)) >= 0) {
            out.write(buf, 0, n);
        }
    }
}
```


## 결론
    - try-finally 말고 try-with-resources를 사용하자 코드가 간결해지고 만들어지는 예외 정보도 유용하다.
    - try-with-resources를 사용하면 정확하고 쉽게 자원을 회수 할 수 있다.
