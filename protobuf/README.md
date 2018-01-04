1、下载 protoc-2.6.1-win32.zip 
https://github.com/google/protobuf/releases?after=v3.0.0-alpha-1
压缩包内有一个protoc.exe，用来把proto的消息文件生成java实体类


2、建立一个Maven工程，添加protobuf-java的依赖。依赖的版本要和上面的protoc的版本保持一致
依赖包地址如下：
http://mvnrepository.com/artifact/com.google.protobuf/protobuf-java/2.6.1
<dependency>  
    <groupId>com.google.protobuf</groupId>  
    <artifactId>protobuf-java</artifactId>  
    <version>2.6.1</version>  
</dependency>  


3、在protoc.exe 目录里建立一个测试的msg.proto文件，内容如下：
option java_package = "com.forever.zhb.protobuf";
option java_outer_classname = "PersonProbuf";
 
message Person {  
 required string name = 1;  
 required int32 id = 2;  
 optional string email = 3;  
  
 enum PhoneType {  
   MOBILE = 0;  
   HOME = 1;  
   WORK = 2;  
 }  
  
 message PhoneNumber {  
   required string number = 1;  
   optional PhoneType type = 2 [default = HOME];  
 }  
  
 repeated PhoneNumber phone = 4;  
}  
  
message AddressBook {  
 repeated Person person = 1;  
} 


4、生成 java文件：打开命令窗口（cmd），进入proto.exe目录下，执行如下命令：
protoc.exe --java_out=C:\code\zhb_forever\src\main\java msg.proto   注意路径根据实际情况修改
会在C:\code\zhb_forever\src\main\java\com\forever\zhb\protobuf包下生成PersonProbuf.java


5、建立测试：
public void testProtobuf(HttpServletRequest request,HttpServletResponse response){
		// 创建builder对象
		PersonProbuf.Person.Builder builder = PersonProbuf.Person.newBuilder();
		builder.setEmail("ggggggg@email.com");
		builder.setId(1);
		builder.setName("TestName");
		builder.addPhone(PersonProbuf.Person.PhoneNumber.newBuilder().setNumber("13664262866")
		.setType(PersonProbuf.Person.PhoneType.MOBILE));
		builder.addPhone(PersonProbuf.Person.PhoneNumber.newBuilder().setNumber("02869387891")
		.setType(PersonProbuf.Person.PhoneType.HOME));
		// 通过builder转化为Person对象
		Person person = builder.build();
		// 将person对象转化为字节流
		byte[] buf = person.toByteArray();
		System.out.println(buf);
		
		// 通过字节流转化 为Person对象
		try {
			Person person2 = PersonProbuf.Person.parseFrom(buf);
			log.info(person2.getName() + ", " + person2.getEmail());
			List<PhoneNumber> lstPhones = person2.getPhoneList();
			for (PhoneNumber phoneNumber : lstPhones) {
				log.info(phoneNumber.getNumber());
			}
		} catch (InvalidProtocolBufferException e) {
			e.printStackTrace();
		} 
	}
