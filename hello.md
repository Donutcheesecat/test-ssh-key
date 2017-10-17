ThoughtWorks2018校园招聘作业题目——出租车
=============
-------------
实现方式：将待检测车辆名单写入txt文件，再从txt文件中逐行读取进行检查车辆信息
# 车辆信息数据结构
```Java
package com.thoughtworks;

import java.time.LocalDate;
/*
 * 车辆信息类
 */
public class CarInfo {
	String plateNum;//车牌号
	LocalDate date;//购买日期
	String brand;//汽车品牌
	int mileage;//⽬目前运⾏行行公⾥里里数
	char overhaul;//有⽆无⼤大修
	public CarInfo(String plateNum,LocalDate date,String brand,int mileage,char overhaul){
		this.plateNum = plateNum;
		this.date = date;
		this.brand = brand;
		this.mileage = mileage;
		this.overhaul = overhaul;
	}
	public boolean isOverhaul(){
		boolean judge = false;
		if(this.overhaul=='T')
			judge = true;
		return judge;
	}
}
```
# 格式化车辆信息
```JAVA
package com.thoughtworks;

import java.time.LocalDate;
/*
 * 由字符串格式化为CarInfo的数据结构
 */
public class FormatCar {
	public static CarInfo getCar(String s){
		String[] arr = s.split("\\|");
    	String plateNum = arr[0];
    	String[] tempDate = arr[1].split("/");
    	LocalDate date = LocalDate.of(Integer.valueOf(tempDate[0]), Integer.valueOf(tempDate[1]), Integer.valueOf(tempDate[2]));
    	String brand = arr[2];
    	int mileage = Integer.valueOf(arr[3]);
    	char overhaul = arr[4].charAt(0);
		
		return new CarInfo(plateNum,date,brand,mileage,overhaul);
	}
}
```
# 判断车辆报废与保养的工具类
```JAVA
package com.thoughtworks;

import java.time.LocalDate;

public class ReminderUtil {
	public static final int F_SERVICE_LIFE = 2190;//无大修的报废周期
	public static final int T_SERVICE_LIFE = 1095;//大修过的报废周期
	public static boolean timeRelated(CarInfo car,LocalDate curDate){//定期保养
		boolean judge = false;
		LocalDate temp = car.date; 
		if(car.isOverhaul()){
			while(!temp.isAfter(curDate)&&!temp.equals(curDate)){
				temp = temp.plusMonths(3);
			}
		}
		else{
			if(curDate.getYear()-car.date.getYear()<3){
				while(!temp.isAfter(curDate)&&!temp.equals(curDate)){
					temp = temp.plusMonths(12);
				}
			}
			else{ 
				while(!temp.isAfter(curDate)&&!temp.equals(curDate)){
					temp = temp.plusMonths(6);
				}
			}
		}
		if(curDate.getMonthValue() == temp.getMonthValue()){
			if(curDate.getDayOfMonth()<=temp.getDayOfMonth())
				judge = true;
		}
		else if( temp.getMonthValue() - curDate.getMonthValue()==1){
			judge = true;
		}
		return judge;
	}
	public static boolean distanceRelated(CarInfo car,LocalDate curDate){//每1万公⾥里里保养
		boolean judge = false;
		if(car.mileage%10000==0||(car.mileage%10000>=9500&&car.mileage%10000<=9999))
			judge = true;
		return judge;
	}
	public static boolean writerOffRelated(CarInfo car,LocalDate curDate){//报废提醒
		boolean judge = false;
		LocalDate writerOffDate = null;
		if(car.isOverhaul())
			writerOffDate = car.date.plusDays(T_SERVICE_LIFE);
		else
			writerOffDate = car.date.plusDays(F_SERVICE_LIFE);
		if(writerOffDate.isAfter(curDate)){
			if(writerOffDate.getYear() == curDate.getYear()){
				if(writerOffDate.getMonthValue()-curDate.getMonthValue()==1)
					judge = true;
				else if(writerOffDate.getMonthValue() == curDate.getMonthValue()){
					if(curDate.getDayOfMonth()<writerOffDate.getDayOfMonth())
						judge = true;
				}

			}
		}
		return judge;
	}
	public static boolean writerOff(CarInfo car,LocalDate curDate){//车辆是否已经报废
		boolean judge = false;
		LocalDate writerOffDate = null;
		if(car.isOverhaul())
			writerOffDate = car.date.plusDays(T_SERVICE_LIFE);
		else
			writerOffDate = car.date.plusDays(F_SERVICE_LIFE);
		if(writerOffDate.isBefore(curDate)||writerOffDate.equals(curDate))
			judge = true;
		return judge;
	}

}
```
# 对读取车辆的检测与分类
```JAVA
package com.thoughtworks;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.TreeMap;

public class Check {
	public static void check(String path){
		TreeMap<String,ArrayList<String>> timeRelated = new TreeMap<>();//报废提醒
		TreeMap<String,ArrayList<String>> distanceRelated = new TreeMap<>();//每1万公⾥里里保养提醒
		TreeMap<String,ArrayList<String>> writerOffRelated = new TreeMap<>();//定期保养
		File file = new File(path);
		BufferedReader reader = null;
		try {
			reader = new BufferedReader(new FileReader(file));
			String tempString = null;
			String[] tempDate = reader.readLine().replaceAll("SubmitDate:","").trim().split("/");
			LocalDate curDate = LocalDate.of(Integer.valueOf(tempDate[0]), Integer.valueOf(tempDate[1]), Integer.valueOf(tempDate[2]));
			while((tempString = reader.readLine())!=null){
				CarInfo car = FormatCar.getCar(tempString);
				if(!ReminderUtil.writerOff(car,curDate)){//判断车辆是否已经报废
					if(ReminderUtil.writerOffRelated(car,curDate)){//判断车辆是否需要报废提醒
						if(!writerOffRelated.containsKey(car.brand)){
							writerOffRelated.put(car.brand, new ArrayList<String>());
							writerOffRelated.get(car.brand).add(car.plateNum);
						}
						else
							writerOffRelated.get(car.brand).add(car.plateNum);
					}
					else{
						if(ReminderUtil.distanceRelated(car,curDate)){//判断车辆是否需要每1万公⾥里里保养
							if(!distanceRelated.containsKey(car.brand)){
								distanceRelated.put(car.brand, new ArrayList<String>());
								distanceRelated.get(car.brand).add(car.plateNum);
							}
							else
								distanceRelated.get(car.brand).add(car.plateNum);
						}
						else if(!ReminderUtil.distanceRelated(car,curDate)&&ReminderUtil.timeRelated(car,curDate)){//判断车辆是否需要定期保养
							if(!timeRelated.containsKey(car.brand)){
								timeRelated.put(car.brand, new ArrayList<String>());
								timeRelated.get(car.brand).add(car.plateNum);
							}
							else
								timeRelated.get(car.brand).add(car.plateNum);
						}
					}
				}
			}
			Print.print(timeRelated, distanceRelated, writerOffRelated);//打印查询结果
			reader.close();	//关闭输入流
		} catch (Exception e) {
			e.printStackTrace();
		}
		finally{

		}
	}
}
```
# 打印工具
```JAVA
package com.thoughtworks;

import java.util.ArrayList;
import java.util.TreeMap;
import java.util.Map.Entry;
/*
 * 查询结果打印
 */
public class Print {
	public static void print(TreeMap<String,ArrayList<String>> timeRelated,TreeMap<String,ArrayList<String>> distanceRelated,TreeMap<String,ArrayList<String>> writerOffRelated){
		System.out.println("Reminder");
		System.out.println("==================");
		System.out.println();
		System.out.println("* Time-related maintenance coming soon...");
		for(Entry<String,ArrayList<String>> entry : timeRelated.entrySet()){
			System.out.print(entry.getKey()+": "+entry.getValue().size()+" (");
			for(int i=0;i<entry.getValue().size();i++){
				if(i!=entry.getValue().size()-1)
					System.out.print(entry.getValue().get(i)+", ");
				else
					System.out.print(entry.getValue().get(i));
			}
			System.out.println(")");

		}
		System.out.println();
		System.out.println("* Distance-related maintenance coming soon...");
		for(Entry<String,ArrayList<String>> entry : distanceRelated.entrySet()){
			System.out.print(entry.getKey()+": "+entry.getValue().size()+" (");
			for(int i=0;i<entry.getValue().size();i++){
				if(i!=entry.getValue().size()-1)
					System.out.print(entry.getValue().get(i)+", ");
				else
					System.out.print(entry.getValue().get(i));
			}
			System.out.println(")");
		}
		System.out.println();
		System.out.println("* Write-off maintenance coming soon...");
		for(Entry<String,ArrayList<String>> entry : writerOffRelated.entrySet()){
			System.out.print(entry.getKey()+": "+entry.getValue().size()+" (");
			for(int i=0;i<entry.getValue().size();i++){
				if(i!=entry.getValue().size()-1)
					System.out.print(entry.getValue().get(i)+", ");
				else
					System.out.print(entry.getValue().get(i));
			}
			System.out.println(")");
		}
	}
}
```
# 测试类
```JAVA
package com.thoughtworks;

public class Test {
	 public static void main(String[] args){
		 Check.check("D:\\text1.txt");//查询入口
		 Check.check("D:\\text2.txt");//查询入口
	 }
}
```
# 测试结果


