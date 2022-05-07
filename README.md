# Logger-of-temperature-and-humidity

![image](https://user-images.githubusercontent.com/19898602/167261056-f8f4d6ea-3add-4fc5-a051-cd9d12607d67.png)


How to measure temperature and humidity. The results are saved in DBF database and shown on 2D chart.



Logger for temperature and humidity

# Used hardware

Numato 8 channel USB GPIO Module;
Connector board for ADC 5V;
AltonaLab temperature sensor;
AltonaLab humidity sensor;
5V multi connector power supply board;
220V to 12V, 2A power supply;

A real image of the connected boards:

![image](https://user-images.githubusercontent.com/19898602/167261073-60544a34-9cab-44e0-a55a-0bb302350d66.png)

AltonaLab diagram is:

![image](https://user-images.githubusercontent.com/19898602/167261081-a1ccac29-93a1-4ca6-95f7-87e8e7112ad0.png)

GPIO 0 and GPIO 1 of Numato board are used as analog inputs. Using the Connector board, the Temperature sensor is connected to GPIO 0 and the Humidity sensor is connected to GPIO 1. The both sensors are supplied with 5V power by 5V power supply board, connected to Connector board.

 

AltonaLab software has a specially developed functional blocks for temperature and humidity measurement: AltonaSensorTemp and AltonaSensorHumidity. These blocks are used in the diagram above:

Using the parameters of these two blocks, we just have to set that the input of the blocks gets raw ADC data from 10bit ADC, which works in range 0..5V and with parameter QueueSize to determine the amount of last measurements over which to calculate the average value of the measured signal:

The output OnRead of the Numato block is connected with Process inputs of AltonaSensorTemp and AltonaSensorHumidity blocks and when the data is read from Numato block, it signals the next blocks to start theirs calculations. The block AltonaSensorHumidity has a temperature compensation - input Temp, so if the temperature of the environment is known, it will change slightly the value of the humidity and will increase its accuracy.

When the diagram is run, to change the temperature, just touch the black sensitive element of the temperature sensor. Because the human temperature is higher, the shown temperature at the running AltonaLab diagram will increase. To increase the measured humidity, just blow on the sensor. Because your breath is moist, the sensor will detect more humidity.



The two sensor blocks AltonaSensorTemp, AltonaSensorHumidity. The outputs of the sensors blocks AltonaSensorTemp and AltonaSensorHumidity are shown with Text controls in Red (temperature) and Blue (humidity) colors.

Block DBFStorage - saves the measured values from sensors in DBF database. This kind of database is perfect for some applications where only one software saves/reads data. The database is very fast. The important parameters of this block are:

FullDBFFileName - here has to be entered the file path and file name of the used DBF. Then when a columns are set in parameter DataSet, the DBF database will be automatically designed and saved to this file;
StoreType - determines the time type of the parameters StoreLast and StoreLastAtMemory, can get the next values: LastRecords, LastSeconds, LastMinutes, LastHours, LastDays;
StoreLast - determines how many last measured values to be saved at the DBF database. For example, if StoreType is LastMinutes and StoreLast is 10, all the records in the DBF older than 10 min, will be deleted automatically. This is a tricky way the contains of the database to be self maintaining and the database size will never become too large;
StoreLastAtMemory - determines how many last measured values to be stored at the block's memory and determines the contains of the output DataSet, which then will be shown in the Image control.
 DataSet - this is a very important parameter, which describes the design of the database. At our case, the database has three columns - Date, which has to be checked as Leading date column and is with Date data type, column Temp - with Analog data type and column Hum with analog type. We can add unlimited amount of columns. When we confirm this parameter, the DBF database will be redesigned with all the needful changes. At our case, the DBF database will consists of three columns: Date, Temp, Hum, and one index over the leading Date column. All the columns will appear as inputs of the block, as it is visible from the diagram, the block DBFStorage contains the inputs Date, Temp, Hum. The input Date is connected with CurrTime block, from where the current time will be gotten.
 
 
 
 
 # Parameters of DBFStorage:
 
 ![image](https://user-images.githubusercontent.com/19898602/167261123-d4bade70-0682-48aa-a134-8deb7a71b364.png)


The DBFStorage block has an input Append, which is connected to the output of the block PulseGenerator, which gets a pulse on every one second. This means, DBFStorage block will add a new record with measured values to the DBF database on every one second. Every time, a new record is added, the output OnReady of the block becomes for a short time to a high level and signals the next block DataSetImage to draw the new contains of the database.

If the AltonaLab diagram is stopped and after a time is started again, the contains of the DBF database will be loaded automatically and the existing measured data will be shown together with the new measured data. An empty hole will be visible between the two parts of the data - an old one and a new one. 

Block DataSetImage - the block is connected to the Image control and shows the contains of the DataSet which is connected to the block's input InDataSet. In our case, this is the contains of the block DBFStorage. The important parameters of the block are:

PeriodTime - determines the type of parameter Last. Can be WholePeriodInDataSet, LastSeconds, LastMinutes, LastHours, LastDays;
Last - determines how many last stored at the input DataSet values to be shown. For example, if the parameter PeriodType is LastMinutes and parameter Last is 10, the DataSetImage block will show the last 10 minutes existing in the dataset gotten from the input InDataSet of the block.
Connected to Image Control - using this parameter the functional block is connected to the available on the diagram Image control;
DataSet - with this most important parameter we can design which columns from the input dataset InDataSet of the block to be shown on the Image control, theirs Min/Max values and dot's colors. In our case, the columns are Date with Date data type, Temp with Analog data type and Hum with Analog data type. The column Temp is set to be shown in range 10...60 in Red color. The column Hum is set to be shown in range 0..100 in Blue color. Date column is from a leading column with Date data type and is used for X axis of the image. The trick here is - at this parameter can be described more or less column than into the input DataSet, which is gotten from DBFStorage, but the block DataSetImage will show only the data, if the column described in DataSetImage exists into the input InDataSet gotten from the previous block.



# Parameters of DataSetImage:

![image](https://user-images.githubusercontent.com/19898602/167261152-b87f891b-ebe2-47ed-a12e-a0f325c16bfb.png)



Some ideas for the experiment:

To see the changes at the graphic, you can touch with fingers the temperature sensor, which will increase the temperature or can blow your breath over the humidity sensor, which will increase the humidity;

Can change the parameters of the block DBFStorage: StoreType to LastDays, StoreLast and StoreLastAtMemory to 7. Then to change the parameters of the block DataSetImage: PeriodType to LastDays and Last to 7. Then change the parameter Value of the block PulseGenerator to 60. The result will be a weekly logger. Always be careful and calculate how much records you expect to be stored at DBF and how much at the memory. Too many stored records are danger. For example, if every second is added a new record at the logger which store and show one week period, the total amount of the records are: 7days*24h*60min*60sec = 604,800 records. This records are too many for the computer memory. We can decrease them when we store one record per minute instead of one record per second. If we save one record per minute, the amount of the records for one week are: 7days*24h*60min = 10,080 records. At this case the system will work better. With one record per minute, we can achieve monthly or annual logger too.

For long period logger, we can make some other optimization as to read the sensor's values on every 20 sec, instead one time per second. This is the parameter CommunicatePerSec of the Numato functional block;

If you planing to order this PCB for your projects so you can consider [JLCPCB.com](https://jlcpcb.com/IAT) because

I always prefer [JLCPCB.com](https://jlcpcb.com/IAT) for my PCB needs, [JLCPCB.com](https://jlcpcb.com/IAT) have best deals for their customers
$2 for 1-4 Layer PCBs, free SMT assembly monthly.

If you seriously need quality PCB quickly in your hand then you must have to try JLCPCB PCB manufacturing service. They have Special offer of $2 for 1-4 Layer PCBs, free SMT assembly monthly. If new user signup today from this link [JLCPCB.com](https://jlcpcb.com/IAT) you will get welcome coupons from JLCPCB.


![image](https://user-images.githubusercontent.com/19898602/159014034-3c9a50c3-61c3-40d2-836d-9cadc2317d33.png)
![image](https://user-images.githubusercontent.com/19898602/164385177-de123350-4a1f-4d0f-9f38-68ed7dbd5a9f.png)



SMT Assembly service of [JLCPCB.com](https://jlcpcb.com/IAT) is cherry on top now get your PCB fully assembled and save your time and money
Select components for your PCB from there Parts Library of 200k+ in-stock components
they are offering $30 valued New User coupons  & $24 SMT coupons every month
$8.00 setup fee, and $0.0017  per joint

Now no need to order components separately for you PCB and get free from stress of soldering them on PCB just try PCB SMT assembly service and get you PCB with components pre assembled and ready for the project


👉 Try PCBA service of [JLCPCB.com](https://jlcpcb.com/IAT) and save your time and money, get PCB ready for project, 200K+ components in library of [JLCPCB.com](https://jlcpcb.com/IAT) as well as 3rd party         parts to choose from. 
    Assembly will support 10M+ parts from Digikey, mouser
    
👉 $30 valued New User coupons 

👉 $24 SMT coupons every month


For more detials & offers please visit [JLCPCB.com](https://jlcpcb.com/IAT)




 
