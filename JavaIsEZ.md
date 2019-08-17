We're given a java class file upon execution it asks for the flag as an argument:
```
$java JavaIsEZ
You need to specify the flag...
$java JavaIsEZ Testflag
noob
```
The fist thing to do is to try and disassemble it, using javap this is the output:
```
javap JavaIsEZ.class
public class JavaIsEZ {
  public static void main(java.lang.String[]);
}
```
A rather empty main function, suggesting that the code is hidden somehow.
We check the bytecode with javap -c 
Here is the interesting part
```
public class JavaIsEZ {
  public static void main(java.lang.String[]);
    Code:
       0: goto          113
       3: ldc           #1                  // String noob
       5: jsr           104
       8: return
       9: pop
      10: ldc           #2                  // String You got it right :P
      12: jsr           104
      15: return
      [...]
```
Java is a stack based machine, and so the code does not contain any registers, a good source for reversing this byte code is 
https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html

The bytecode i quite short so reversing the entire thing is not infeasible, but making some highlevel observations might make things quicker.
I had no luck in debugging it, and so i did it by static analysis only.

At some point we see javap give us these comments:
```
278: invokevirtual #26                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
281: ldc           #6                  // String +?>!‼|CU↑o*♣M;j\ts,=♣M;↑☼s*h,J↨F0s?8↕#{Z:
283: invokevirtual #27                 // Method java/lang/String.equals:(Ljava/lang/Object;)Z
```
ldc pushes the value from the constant pool to the stack, and it happens to be a string, or at the least string like.
Then a method of String.equals is called, a string comparison. This is between some string and the constant, if the result is not zero
a jump is made to instruction 9 which loads and persumably prints "You got it right :P".

So far so good, that string compare needs to pass.
Another interesting sub procedure is called right before the above snippet, and pushes a bunch of variables into a int array:
```
30: astore_1          
31: iconst_0          
32: istore        7   
34: bipush        8   
36: newarray       int
38: astore        8   
40: bipush        97  
42: jsr           16  
45: bipush        71  
47: jsr           16  
50: bipush        94  
52: jsr           16  
55: bipush        89  
57: jsr           16  
60: bipush        90  
62: jsr           16  
65: bipush        121 
67: jsr           16  
70: bipush        72  
72: jsr           16  
75: bipush        53  
77: jsr           16  
80: ret           1   
```
It then builds it into a String array.
After this a jump is made to 295, containing this sub procedure:
```
 295: pop                                                                                                     
 296: invokestatic  #18                 // Method java/util/concurrent/ThreadLocalRandom.current:()Ljava/util/
rrent/ThreadLocalRandom;                                                                                      
 299: iconst_1                                                                                                
 300: ldc           #4                  // int 65535                                                          
 302: invokevirtual #19                 // Method java/util/concurrent/ThreadLocalRandom.nextInt:(II)I        
 305: istore        9                                                                                         
 307: iconst_0                                                                                                
 308: istore        10                                                                                        
 310: aload         4                                                                                         
 312: monitorexit                                                                                             
 313: aload         12                                                                                        
 315: aload         4                                                                                         
 317: iload         10                                                                                        
 319: caload                                                                                                  
 320: aload         8                                                                                         
 322: dup                                                                                                     
 323: iload         10                                                                                        
 325: swap                                                                                                    
 326: arraylength                                                                                             
 327: irem                                                                                                    
 328: iaload                                                                                                  
 329: ixor                                                                                                    
 330: i2c                                                                                                     
 331: invokevirtual #28                 // Method java/lang/StringBuilder.append:(C)Ljava/lang/StringBuilder; 
 334: pop                                                                                                     
 335: iinc          10, 1                                                                                     
 338: iconst_0                                                                                                
 339: istore        9                                                                                         
 341: iload         13                                                                                        
 343: ifne          113                                                                                       
 346: goto          310                                                                                       
 349: return                                                                                                  
```
In short this is code to xor your input string with the built string from before.
flag{j4v4_1s_4s_h4rd_4s-n4t1v3_sQ4aaHZ3of}
