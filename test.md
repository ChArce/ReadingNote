## JAVA ���˼��

### ��14�� ������Ϣ


Class����ֻ����Ҫ��ʱ��Ż���أ�static�ĳ�ʼ��������ص�ʱ�����.


�������ʱ��Ҫʹ��������Ϣ������Ҫ��ö�ǡ����Class��������ã�Class.forName()���Բ���ҪΪ�˻��
Class���ö����и����͵Ķ��󡣵�Ȼ�����ʱ����һ������Ȥ�����͵Ķ�����ô����ʹ��getClass()������
��ȡClass����.


```java
Class t = Class.forName(T);
t.newInstance();
```

```java
T t = new T();
```
�����������������Ч����һ���ӵģ����ǻ��Ʋ�һ��
   1. newInstance()���뱣֤���Ѿ����ز����Ѿ����ӣ���new����û�б�����
   2. newInstance()ֻ�ܵ����޲εĹ��캯������new���ɶ���û��������ơ�

���⻹��һ�ַ��������ɶ�Class���������,�� `FancyToy.class`, ���ַ����ڱ���ʱ�ͻ��ܵ���飬Ҳ����Ч��
���ڻ����������ͣ�����һ����׼�ֶ�TYPE����:
| int.class | Integer.TYPE |
| byte.class | Byte.TYPE |


```java
Class<? extends Number> bounded = int.class;
bounded = double.class;
bounded = Number.class;
```

���ڷ�����˵ newInstance()�ǿ��Է��ظö����ȷ�����͵ģ���������е��ǳ��ۣ���ô��ʱnewInstance()���صĲ��Ǿ�ȷ���ͣ���ֻ��Object

