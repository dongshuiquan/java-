# oracle存储过程初学实例

原文链接：<https://blog.csdn.net/qq_37057095/article/details/76669489>



## 认识存储过程和函数 

​	存储过程和函数也是一种PL/SQL块，是存入数据库的PL/SQL块。但存储过程和函数不同于已经介绍过的PL/SQL程序，我们通常把PL/SQL程序称为无名块，而存储过程和函数是以命名的方式存储于数据库中的。和PL/SQL程序相比，存储过程有很多优点，具体归纳如下：

- 存储过程和函数以命名的数据库对象形式存储于数据库当中。存储在数据库中的优点是很明显的，因为代码不保存在本地，用户可以在任何客户机上登录到数据库，并调用或修改代码。
- 存储过程和函数可由数据库提供安全保证，要想使用存储过程和函数，需要有存储过程和函数的所有者的授权，只有被授权的用户或创建者本身才能执行存储过程或调用函数。 
- 存储过程和函数的信息是写入数据字典的，所以存储过程可以看作是一个公用模块，用户编写的PL/SQL程序或其他存储过程都可以调用它(但存储过程和函数不能调用PL/SQL程序)。一个重复使用的功能，可以设计成为存储过程，比如：显示一张工资统计表，可以设计成为存储过程；一个经常调用的计算，可以设计成为存储函数；根据雇员编号返回雇员的姓名，可以设计成存储函数。
- 像其他高级语言的过程和函数一样，可以传递参数给存储过程或函数，参数的传递也有多种方式。存储过程可以有返回值，也可以没有返回值，存储过程的返回值必须通过参数带回；函数有一定的数据类型，像其他的标准函数一样，我们可以通过对函数名的调用返回函数值。 存储过程和函数需要进行编译，以排除语法错误，只有编译通过才能调用。



## 创建和删除存储过程 

​	创建存储过程，需要有CREATE PROCEDURE或CREATE ANY PROCEDURE的系统权限。该权限可由系统管理员授予。创建一个存储过程的基本语句如下：
CREATE [OR REPLACE] PROCEDURE 存储过程名[(参数[IN|OUT|IN OUT] 数据类型...)] 
{AS|IS} 
[说明部分] 
BEGIN 
可执行部分 
[EXCEPTION 
错误处理部分] 
END [过程名]; 

其中： 
可选关键字OR REPLACE 表示如果存储过程已经存在，则用新的存储过程覆盖，通常用于存储过程的重建。 
参数部分用于定义多个参数(如果没有参数，就可以省略)。参数有三种形式：IN、OUT和IN OUT。如果没有指明参数的形式，则默认为IN。 
关键字AS也可以写成IS，后跟过程的说明部分，可以在此定义过程的局部变量。 
编写存储过程可以使用任何文本编辑器或直接在SQL*Plus环境下进行，编写好的存储过程必须要在SQL*Plus环境下进行编译，生成编译代码，原代码和编译代码在编译过程中都会被存入数据库。编译成功的存储过程就可以在Oracle环境下进行调用了。
一个存储过程在不需要时可以删除。删除存储过程的人是过程的创建者或者拥有DROP ANY PROCEDURE系统权限的人。删除存储过程的语法如下： 
DROP PROCEDURE 存储过程名； 
如果要重新编译一个存储过程，则只能是过程的创建者或者拥有ALTER ANY PROCEDURE系统权限的人。语法如下： 
ALTER PROCEDURE 存储过程名 COMPILE； 
执行(或调用)存储过程的人是过程的创建者或是拥有EXECUTE ANY PROCEDURE系统权限的人或是被拥有者授予EXECUTE权限的人。执行的方法如下： 
方法1： 
EXECUTE 模式名.存储过程名[(参数...)]; 
方法2： 
BEGIN 
模式名.存储过程名[(参数...)]; 
END; 
传递的参数必须与定义的参数类型、个数和顺序一致(如果参数定义了默认值，则调用时可以省略参数)。参数可以是变量、常量或表达式，用法参见下一节。 
如果是调用本账户下的存储过程，则模式名可以省略。要调用其他账户编写的存储过程，则模式名必须要添加。 

以下是生成和调用简单存储过程的训练。注意要事先授予创建存储过程的权限。



### 创建一个显示雇员总人数的存储过程

步骤1：登录SCOTT账户(或学生个人账户)。 
步骤2：在SQL*Plus输入区中，输入以下存储过程： 

~~~sql
CREATE 
	OR REPLACE PROCEDURE EMP_COUNT AS V_TOTAL NUMBER ( 10 );
BEGIN
SELECT
	COUNT( * ) INTO V_TOTAL 
FROM
	EMP;
DBMS_OUTPUT.PUT_LINE ( '雇员总人数为：' || V_TOTAL );

END; 
~~~

步骤3：按“执行”按钮进行编译。 
	如果存在错误，就会显示: 
	警告: 创建的过程带有编译错误。 
	如果存在错误，对脚本进行修改，直到没有错误产生。 

步骤4：调用存储过程

​	显示结果为：  

~~~sql
雇员总人数为：14  
~~~



### 编写显示雇员信息的存储过程EMP_LIST，并引用EMP_COUNT存储过程。

步骤1：在SQL*Plus输入区中输入并编译以下存储过程： 

~~~sql
CREATE 
	OR REPLACE PROCEDURE EMP_LIST AS CURSOR emp_cursor IS SELECT
	empno,
	ename 
FROM
	emp;
BEGIN
	FOR Emp_record IN emp_cursor
	LOOP
	DBMS_OUTPUT.PUT_LINE ( Emp_record.empno || Emp_record.ename );

END LOOP;
EMP_COUNT;

END; 
~~~

步骤2：调用存储过程，在输入区中输入以下语句并执行：

显示结果为：  

~~~sql
7369SMITH  
7499ALLEN  
7521WARD  
7566JONES  
            执行结果：  
        雇员总人数为：14  
        PL/SQL 过程已成功完成。 
~~~

说明：以上的EMP_LIST存储过程中定义并使用了游标，用来循环显示所有雇员的信息。然后调用已经成功编译的存储过程EMP_COUNT，用来附加显示雇员总人数。通过EXECUTE命令来执行EMP_LIST存储过程。



### 参数传递 

参数的作用是向存储过程传递数据，或从存储过程获得返回结果。正确的使用参数可以大大增加存储过程的灵活性和通用性。 
参数的类型有三种，如下所示。 

~~~sql
IN  定义一个输入参数变量，用于传递参数给存储过程  
OUT 定义一个输出参数变量，用于从存储过程获取数据  
IN OUT  定义一个输入、输出参数变量，兼有以上两者的功能  
~~~

参数的定义形式和作用如下： 

- 参数名 IN 数据类型 DEFAULT 值； 
  	定义一个 输入参数变量，用于传递参数给存储过程。在调用存储过程时，主程序的实际参数可以是常量、有值变量或表达式等。DEFAULT 关键字为可选项，用来设定参数的默认值。如果在调用存储过程时不指明参数，则参数变量取默认值。在存储过程中，输入变量接收主程序传递的值，但不能对其进行赋值。
- 参数名 OUT 数据类型； 
  	定义一个输出参数变量，用于从存储过程获取数据，即变量从存储过程中返回值给主程序。 
  在调用存储过程时，主程序的实际参数只能是一个变量，而不能是常量或表达式。在存储过程中，参数变量只能被赋值而不能将其用于赋值，在存储过程中必须给输出变量至少赋值一次。	
- 参数名 IN OUT 数据类型 DEFAULT 值； 
  	定义一个输入、输出参数变量，兼有以上两者的功能。在调用存储过程时，主程序的实际参数只能是一个变量，而不能是常量或表达式。DEFAULT 关键字为可选项，用来设定参数的默认值。在存储过程中，变量接收主程序传递的值，同时可以参加赋值运算，也可以对其进行赋值。在存储过程中必须给变量至少赋值一次。
- 如果省略IN、OUT或IN OUT，则默认模式是IN。 



###  编写给雇员增加工资的存储过程CHANGE_SALARY，通过IN类型的参数传递要增加工资的雇员编号和增加的工资额

步骤1：输入以下存储过程并执行：

~~~sql
CREATE 
	OR REPLACE PROCEDURE CHANGE_SALARY ( P_EMPNO IN NUMBER DEFAULT 7788, P_RAISE NUMBER DEFAULT 10 ) AS V_ENAME VARCHAR2 ( 10 );
V_SAL NUMBER ( 5 );
BEGIN
SELECT
	ENAME,
	SAL INTO V_ENAME,
	V_SAL 
FROM
	EMP 
WHERE
	EMPNO = P_EMPNO;
UPDATE EMP 
SET SAL = SAL + P_RAISE 
WHERE
	EMPNO = P_EMPNO;
DBMS_OUTPUT.PUT_LINE (
'雇员' || V_ENAME || '的工资被改为' || TO_CHAR( V_SAL + P_RAISE ));
COMMIT;
EXCEPTION 
	WHEN OTHERS THEN
	DBMS_OUTPUT.PUT_LINE ( '发生错误，修改失败！' );
ROLLBACK;

END;
~~~

说明：从执行结果可以看到，雇员SCOTT的工资已由原来的3000改为3080。 
	参数的值由调用者传递，传递的参数的个数、类型和顺序应该和定义的一致。如果顺序不一致，可以采用以下调用方法。如上例，执行语句可以改为： 
 EXECUTE CHANGE_SALARY(P_RAISE=>80,P_EMPNO=>7788); 
  可以看出传递参数的顺序发生了变化，并且明确指出了参数名和要传递的值，=>运算符左侧是参数名，右侧是参数表达式，这种赋值方法的意义较清楚。 



###  使用OUT类型的参数返回存储过程的结果

步骤1：输入区中输入并编译以下存储过程

~~~sql
CREATE 
	OR REPLACE PROCEDURE EMP_COUNT ( P_TOTAL OUT NUMBER ) AS BEGIN
SELECT
	COUNT( * ) INTO P_TOTAL 
FROM
	EMP;

END;
~~~

说明：在存储过程中定义了OUT类型的参数P_TOTAL，在主程序调用该存储过程时，传递了参数V_EMPCOUNT。在存储过程中的SELECT...INTO...语句中对P_TOTAL进行赋值，赋值结果由V_EMPCOUNT变量带回给主程序并显示。
以上程序要覆盖同名的EMP_COUNT存储过程，如果不使用OR REPLACE选项，就会出现以下错误：

~~~sql
ERROR 位于第 1 行:  
        ORA-00955: 名称已由现有对象使用。  
~~~

### 使用IN OUT类型的参数，给电话号码增加区码

步骤1：输入并编译以下存储过程： 

~~~sql
CREATE 
	OR REPLACE PROCEDURE ADD_REGION ( P_HPONE_NUM IN OUT VARCHAR2 ) AS BEGIN
	P_HPONE_NUM := '0755-' || P_HPONE_NUM;

END;
~~~

说明：变量V_HPONE_NUM既用来向存储过程传递旧电话号码，也用来向主程序返回新号码。新的号码在原来基础上增加了区号0755和-。

