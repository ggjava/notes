
反转字符串意味着按相反的顺序排列字符串的字符，也就是说，将最后一个字符放在第一位，将第二个最后一个字符放在第二位，以此类推。
本文将通过程序示例讨论在java中反转字符串的不同方法。

## StringBuffer

java.lang.StringBuffer类有一个反转方法，该方法反转提供给它的字符串。

```java
public class StringReverseDemo {
    public static void main(String[] args) {
        String str = "linuxcoming";
        // create an object of stringbuffer
        StringBuffer buffer = new StringBuffer(str);
        System.out.println("Original string: " + str);
        System.out.println("Reversed string: " + buffer.reverse());
   }
}
```

输出：

```
Original string: linuxcoming
Reversed string: gnimocxunil
```

> 还可以使用 java.lang.StringBuilder，string类中没有reverse函数。

## char数组

```java
public class StringReverseDemo {
    public static void main(String[] args) {
        String str = "linuxcoming";
        // create a new character array of string length
        char[] copyArray = new char[str.length()];
        // get characters of source string
        char[] stringChars = str.toCharArray();
        int startIndex = 0;
        // iterate over the string characters backwards
        for(int i = stringChars.length -1; i >=0; i--) {
           // store characters in the copy array
           copyArray[startIndex] = stringChars[i];
           // increment the index of copy array
           startIndex++;
        }
        // create a new string from character copy
        String reverse = new String(copyArray);
        System.out.println("Original string: " + str);
        System.out.println("Reversed string: " + reverse);
    }
}
```

如果要求您在不使用任何字符串函数的情况下反转字符串，那么这就是要学习的方法。此方法基于以下算法

* 创建与字符串中的字符数大小相同的新字符数组。
* 将源字符串转换为字符数组。
* 向后遍历字符。
* 在每个迭代中，将当前字符添加到步骤2中创建的新数组中。这个字符数组不会按相反的顺序保存字符。
* 使用步骤4中填充的数组创建一个新字符串。

> 上面的代码是按照前面提到的算法编写的。要将字符数组转换为字符串，请使用java.lang.String的构造函数。

在此方法中，还可以使用字节数组而不是字符数组。要将字符串转换为字节数组，请使用string的getBytes方法。要将字节数组转换为字符串，请使用字符串类构造函数，该构造函数接受字节数组作为参数。

## char数组+StringBuffer

这个方法是上面方法1和方法2的组合。在这个方法中，我们还将字符串转换为字符数组，并以相反的顺序遍历它。

它不是将字符放入字符数组，而是添加到java.lang.StringBuffer 使用其append方法的StringBuffer对象，该方法使用toString方法转换为字符串。

```java
public class StringReverseDemo {
    public static void main(String[] args) {
        String str = "codippa";
        // create a stringbuffer to hold the reversed string
        StringBuffer buffer = new StringBuffer();
        // get characters of source string
        char[] stringChars = str.toCharArray();
        // iterate over the string characters backwards
        for(int i = stringChars.length -1; i >=0; i--) {
           // add character to stringbuffer
           buffer.append(stringChars[i]);
        }
        // convert stringbuffer to string
        String reverse = buffer.toString();
        System.out.println("Original string: " + str);
        System.out.println("Reversed string: " + reverse);
   }
}
```

## ArrayList

```java
public class StringReverseDemo {
    public static void main(String[] args) {
        String s = "codippa";
        // get characters of source string
        char[] stringChars = s.toCharArray();
        // list to hold the characters
        List list = new ArrayList();
        // iterate over the string characters
        for(int i = 0; i < stringChars.length; i++) {
           // add character to list
           list.add(stringChars[i]);
        }
        // reverse the list elements
        Collections.reverse(list);
        for (int i = 0; i < list.size(); i++) {
           System.out.print(list.get(i));
        }
    }
}
```

此方法将遵循以下步骤。

* 将字符串转换为字符数组。
* 初始化一个java.util.ArrayList。
* 遍历字符数组，在每次迭代中，将当前字符添加到创建的List中。
* 使用java.util.Collections的reverse方法对数组列表中的元素进行反向。
* 遍历数组列表并在每次迭代中打印当前元素。

如果您想要生成字符串，不是仅仅打印相反的字符，只需这些字符添加到java.lang.StringBuffer或字符数组，就像我们在前面的方法中所做的那样。

> 此方法不推荐使用，仅用于学习目的。它的效率非常低，因为它涉及到对字符串的字符进行两次迭代。因此，对于较大的字符串，与其他方法相比，它将非常慢。

## 字符交换

该方法通过交换字符串两端的字符来反转字符串。也就是说，第一个字符与最后一个字符交换，第二个字符与第二个字符交换，以此类推。
为此，初始化两个索引:一个在最左边的位置，另一个在最右边的位置。然后迭代，直到开始(或左)索引小于结束(或右)索引。
在每次迭代中，交换字符。当开始索引与结束索引相等时，这意味着覆盖了所有字符，循环将终止。

```java
public class StringReverseDemo {
    public static void main(String[] args) {
        String str = "linuxcoming";
        // get characters of source string
        char[] stringChars = str.toCharArray();
        // initialize start and end indexes
        int start = 0; 
        int end = stringChars.length - 1;
        // iterate over the character array
        while(start < end) {
           // swap start and end characters
           char temp = stringChars[end];
           stringChars[end] = stringChars[start];
           stringChars[start] = temp;
           // increment start index
           start++;
           // reduce end index
           end--;
        }
        // create a new string with character array
        String reverse = new String(stringChars);
        System.out.println("Original string: " + str);
        System.out.println("Reversed string: " + reverse);
    }
}
```

## 递归

字符串也可以使用递归进行反转。

要通过递归反转字符串，创建一个接受字符串和索引位置的方法。一次又一次调用此方法，每次索引都减少1。

在每次调用此方法时，它都将在提供的索引位置提取字符，并将其添加到开头，后面是通过使用索引减少1的方法调用此方法返回的字符串。

首次调用此方法时，索引为（字符串长度–1）。因此，它将提取最后一个字符并将其添加到开头。

当第二次调用时，它将提取第二个字符并将其添加到开头。当索引变为0时，它将返回第一个字符。

因此，当索引变为0时，返回的字符串将与原始字符串相反。

```java
public class StringReverseDemo {
    public static void main(String[] args) {
        String str = "linuxcoming";
        // call recursive method with index of the last character
        String reverse = reverse(str, str.length() - 1);
        System.out.println("Original string: " + str);
        System.out.println("Reversed string: " + reverse);
    }

    /**
    * Recursive method to reverse a string
    * @param stringToReverse
    * @param position
    * @return
    */
    static String reverse(String stringToReverse, int position) {
        // index is zero, means return the first character
        if (position == 0) {
               return "" + stringToReverse.charAt(0);
        }
        // get character at supplied index
        char characterAtIndex = stringToReverse.charAt(position);
        // string with character at right end placed towards the left
        return characterAtIndex + reverse(stringToReverse, position - 1);
   }
}
```

> 如果您被要求使用for循环在java中反转字符串，那么使用方法2和5，因为它们不涉及任何string类函数。
