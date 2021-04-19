# Project 6
- **Assigned: Friday April 23, 2021**
- **Due: Friday May 7, 2021 at 11:59pm**
---
### Messaging and the Chandy-Lamport Snapshot Algorithm

This project has three tasks:
-	Add the Chandy-Lamport Snapshot Algorithm into a distributed system.
-	Use the Chandy-Lamport Snapshot to understand the behavior of a system.
-	Add monopoly-seeking behavior to the players in a simulation.

The distributed system is loosely based on the card game *Pit*. (See [Wikipedia](https://en.wikipedia.org/wiki/Pit_(game)) or [Amazon](https://amazon.com/Winning-Moves-Games-Deluxe-Pit/dp/B00000DMBD))

Our simulation of the game has five players. Each player has a set of commodities that it trades with the other players. A player initiates a trade by making an offer to another randomly chosen player. The other player can accept or reject the trade. If they accept it, they pay with a commodity of their own. If they reject it, they send back the commodity. The trading action therefore is a fast series of offers, acceptances, rejections, and more offers.

Each of the 5 players is modeled as a Message Driven Bean. The code for each is nearly identical, except for its class name, the Queue it listens to, and an instance variable named myPlayerNumber. Each of the players instantiates a PITPlayerModel which does all the business (game) logic for the simulation.

All communication between the players is done by JMS Message Queues. Each player listens to its own Queue. Other players can communicate with the player by sending a message to its Queue.  
![Players and Queues](/images/PlayerQueues.png "Players and Queues")

A servlet and a Test Snapshot web page allow the system to be tested. Clicking on the Start Simulation button will start the simulation running. The servlet will send a series of messages to each Player's Queue.

- First it sends a Reset.HALT message to each Player and awaits its acknowledgement response. This ensures that the players stop trading if they had been actively doing so.
- Next it sends a Reset.CLEAR message to each Player to have them reset their data structures and awaits their responses.
- Once all five Players have been reset, it sends a NewHand message to each with a set of commodities. In this way, each Player is assigned its own initial set of commodities. These commodities are also known as cards. A Player begins trading as soon as it receives its NewHand.

Trading continues until the maxTrades threshold is hit. This can be adjusted in the PITPlayerModel so the trading does not go on forever. The trading can also be stopped by clicking the Halt Simulation button on the Test Snapshot page.

A new round of trading can then be started by using the PITsnapshot servlet again.

Initially, the Test Snapshot page will show a list of Snapshot Failed messages (see picture below). This is normal because the snapshot has not yet been implemented. You will be implementing it.

![Failing snapshots](/images/FailingSnapshots.png)

### Setting up the ConnectionFactory and Queues

Unlike the JMS Lab, you will need to set up the ConnectionFactory and Queues outside the program itself. This can be done by editing the `tomee.xml` file. The `tomee.xml` file is within the `conf` directory of the `apache-tomee-plus-version` directory. For example: `apache-tomee-plus-8.0.4/conf/tomee.xml`

[This gist](https://gist.github.com/joemertz/b7d0a88cb65022a02fbbc4a9d46bea7d) is a `tomee.xml` file with the resources necessary to add for Project 6 clearly indicated.

Insert into your own `tomee.xml` file the [gist](https://gist.github.com/joemertz/b7d0a88cb65022a02fbbc4a9d46bea7d) code shown between the

` <!-- BEGIN: Add for Project 6 -->`  
and  
`<!-- END: for Project 6 --> `

### Installing the system
 1. From this GitHub repository, click the "Clone or download" button and copy the repository URL.
 2. Start IntelliJ, and from the "Welcome to IntelliJ IDEA" window, choose "Get from Version Control"
 3. Paste the GitHub repository URL into the dialog box and select "Clone".

(You can ignore the "Invalid VCS root mapping" message if you get one.)

 4. Create a Run/Debug Configuration  
 a. Click "+" to add a "TomEE Server" of type "Local"  
 b. Configure the Application Server (See Lab 1 for instructions)  
 c. Set VM options to: `-Dorg.apache.activemq.SERIALIZABLE_PACKAGES=*`  
 d. In the Deployment Tab, add the following artifacts:
    - PITsimulation:ejb exploded  
    - PITdashboard:war exploded  

 e. Select the "PITdashboard:war exploded" and set the Application Context to "/"  
 f. In the Logs Tab, select the checkbox "Save console output to file:" and provide a path and file name for where the output can be saved. In addition to being shown in IntelliJ, your console messages (i.e. System.out.print..) will be saved to this file.    
 g. Click OK to safe the configuration.

Run TomEE and the PITdashboard (a web application) and the PITsimulation (a set of MDBs) should be deployed.

### Testing
Open a web browser and browse to the URL:
[http://localhost:8080/](http://localhost:8080/)

Click on the button to start the simulation and after a short while you will see the response: "PIT has been initiated"

Go to the Server Log in IntelliJ and review the output that is produced by the system. It should look something like the following.

(Note: there will be many more lines before this, and these lines will likely scroll past fast and be lost from the console history. Therefore view the console log file that you set up in step 4f above.)
```
Servlet sending Reset HALT to PITplayer0
PITplayer0:  received Reset HALT
Servlet Reset HALT from PITplayer0 ACKNOWLEDGED
Servlet sending Reset HALT to PITplayer1
PITplayer1:  received Reset HALT
Servlet Reset HALT from PITplayer1 ACKNOWLEDGED
Servlet sending Reset HALT to PITplayer2
PITplayer2:  received Reset HALT
Servlet Reset HALT from PITplayer2 ACKNOWLEDGED
Servlet sending Reset HALT to PITplayer3
PITplayer3:  received Reset HALT
Servlet Reset HALT from PITplayer3 ACKNOWLEDGED
Servlet sending Reset HALT to PITplayer4
PITplayer4:  received Reset HALT
Servlet Reset HALT from PITplayer4 ACKNOWLEDGED
Servlet sending Reset CLEAR to PITplayer0
PITplayer0:  received Reset CLEAR
Servlet Reset CLEAR from PITplayer0 ACKNOWLEDGED
Servlet sending Reset CLEAR to PITplayer1
PITplayer1:  received Reset CLEAR
Servlet Reset CLEAR from PITplayer1 ACKNOWLEDGED
Servlet sending Reset CLEAR to PITplayer2
PITplayer2:  received Reset CLEAR
Servlet Reset CLEAR from PITplayer2 ACKNOWLEDGED
Servlet sending Reset CLEAR to PITplayer3
PITplayer3:  received Reset CLEAR
Servlet Reset CLEAR from PITplayer3 ACKNOWLEDGED
Servlet sending Reset CLEAR to PITplayer4
PITplayer4:  received Reset CLEAR
Servlet Reset CLEAR from PITplayer4 ACKNOWLEDGED
Servlet sending newhand to 0
Servlet sending newhand to 1
PITplayer0:  numTrades: 0
PITplayer0:  new hand: size: 15 Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer0:  offered: Platinum to player: 4
PITplayer1:  numTrades: 0
PITplayer1:  new hand: size: 15 Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer1:  offered: Platinum to player: 0
Servlet sending newhand to 2
PITplayer2:  numTrades: 0
PITplayer2:  new hand: size: 15 Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer2:  offered: Platinum to player: 1
Servlet sending newhand to 3
PITplayer3:  numTrades: 0
PITplayer3:  new hand: size: 15 Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer3:  offered: Platinum to player: 1
Servlet sending newhand to 4
PITplayer4:  numTrades: 0
PITplayer4:  new hand: size: 15 Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer4:  offered: Platinum to player: 3
PITplayer0:  received offer of: Platinum from player: 1
PITplayer0:  accepting offer and paying with: Palladium to player: 1
PITplayer0:  hand: size: 14 Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum
PITplayer1:  received offer of: Platinum from player: 2
PITplayer1:  rejecting offer of: Platinum from player: 2
PITplayer1:  hand: size: 14 Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer4:  received offer of: Platinum from player: 0
PITplayer4:  rejecting offer of: Platinum from player: 0
PITplayer4:  hand: size: 14 Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium
PITplayer3:  received offer of: Platinum from player: 4
PITplayer3:  accepting offer and paying with: Palladium to player: 4
PITplayer3:  hand: size: 14 Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum
PITplayer1:  received offer of: Platinum from player: 3
PITplayer1:  accepting offer and paying with: Palladium to player: 3
PITplayer1:  hand: size: 14 Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum
PITplayer0:  received rejected offer of: Platinum from player: 4
PITplayer0:  hand: size: 15 Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Platinum
PITplayer0:  offered: Iridium to player: 3
PITplayer2:  received rejected offer of: Platinum from player: 1
PITplayer2:  hand: size: 15 Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum
PITplayer2:  offered: Palladium to player: 3
PITplayer4:  received: Palladium as payment from player: 3
PITplayer4:  hand: size: 15 Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Palladium
PITplayer4:  offered: Palladium to player: 1
PITplayer1:  received: Palladium as payment from player: 0
PITplayer1:  hand: size: 15 Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium
PITplayer1:  offered: Iridium to player: 2
PITplayer3:  received offer of: Palladium from player: 2
PITplayer3:  accepting offer and paying with: Iridium to player: 2
PITplayer3:  hand: size: 14 Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium
PITplayer1:  received offer of: Palladium from player: 4
PITplayer1:  rejecting offer of: Palladium from player: 4
PITplayer1:  hand: size: 14 Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium
PITplayer3:  received offer of: Iridium from player: 0
PITplayer3:  accepting offer and paying with: Rhodium to player: 0
PITplayer3:  hand: size: 14 Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium
PITplayer2:  received offer of: Iridium from player: 1
PITplayer2:  accepting offer and paying with: Iridium to player: 1
PITplayer2:  hand: size: 14 Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Iridium
PITplayer3:  received: Palladium as payment from player: 1
PITplayer3:  hand: size: 15 Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Palladium
PITplayer3:  offered: Ruthenium to player: 0
PITplayer2:  received: Iridium as payment from player: 3
PITplayer2:  hand: size: 15 Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Iridium Iridium
PITplayer2:  offered: Rhodium to player: 1
PITplayer1:  received: Iridium as payment from player: 2
PITplayer0:  received: Rhodium as payment from player: 3
PITplayer0:  hand: size: 15 Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Palladium Iridium Rhodium Ruthenium Platinum Platinum Rhodium
PITplayer0:  offered: Rhodium to player: 4
PITplayer4:  received rejected offer of: Palladium from player: 1
```
This is a global history of the actions being taken by the 5 players.  It will eventually stop when each Player hits 20000 trades or you click Halt Simulation.

In the global history will be lines similar to:
```
Servlet Initiating Snapshot via PITplayer4
PITplayer4 received unknown Message type
Servlet: Timeout number 1 without a player reporting.
```

The first message is from the Servlet indicating that it is about to send a marker message into the queue of one of the PITplayers. PITplayer4 then reports that it got a message of unknown type (because it is of type Marker and it doesn’t know how to handle them (yet)). The final line is from the Servlet again reporting that it has not received snapshot messages back from all of the players. At this point these console messages make sense because you have not implemented the snapshot algorithm yet.

Back in the browser, snapshot test results will be added to the window. They will all be failing, so the results will look like the screenshot above. The snapshots are failing because the snapshot code has not yet been implemented. That is your task; implement the snapshot code.

The Test Snapshot web page is reusable without re-loading. (It uses AJAX.) So, at any time you can just click on Start Simulation to restart the next set of snapshots.

Do not use Internet Explorer to test with the Test Snapshot page. IE erroneously caches the AJAX requests and you will get invalid results. Use Chrome or Safari instead.

 :checkered_flag:
 **When you get the simulation to hit 20000 trades, you have completed the Commodity Trading Simulation lab. Show a TA for credit, and you are now ready to start the project.**

#### Managing Large Console Logs
Messages logged to the console act as the process histories of the players and the global history of the whole distributed system. The message are critical to debugging, but not all messages are useful all-the-time.  Therefore it is advisable to comment out log messages that you will not need.

The log messages are printed to the console very quickly and the IntelliJ console will only keep a limited history.  Therefore the console will quickly lose your initial logging messages.  Here are two hints to be able to examine the full log history:
1. Turn off informational (unsevere) activemq and geronimo messages:  
In the same TomEE `conf` directory that you found the `tomee.xml` file to edit earlier, there is another file named `logging.properties`.  Edit that file and **uncomment** the two lines by removing the #:  
org.apache.activemq.level = SEVERE  
org.apache.geronimo.level = SEVERE  
2. View the console log file that you set up in step 4f above. Note that these files can be very big and not all editors can handle them well. The open source [Atom](https://atom.io/) text editor seems to do very well.

### Task – Implement the Chandy Lamport Snapshot Algorithm
In class, we discuss the Chandy Lamport Snapshot Algorithm. Implement this algorithm in the system so that you can check the state of players and commodities, such as what player is holding what commodities. Since there are 15 of each commodity given out by PITsnapshot, and none or consumed or added, there should always be a steady state of 15 of each commodity shared between the 5 Players. (Each Player, however, may have more or less than 15 commodities at any given time.)

Some pieces have been provided to you for this task:

The Marker class is defined for passing as the Marker in the snapshot algorithm.

The servlet PITsnapshot will initiate the snapshot by sending a Marker to some player. (All 5 Players run the same PITPlayeModel code, so the PITsnapshot should be able to initiate the snapshot by sending to any of the 5 Players.)

PITsnapshot will then wait and read from the PITsnapshot Queue. Each Player should send a message back to PITsnapshot via that Queue. The content of that message should be an ObjectMessage, and the Object should be a HashMap of commodities and counts (see the code for details). Add to the HashMap the identify of who the snapshot is coming from in the format: state.put("Player", myPlayerNumber);

Finally, the PITsnapshot servlet will report the sums of each commodity back to the browser.

The picture below shows only the first two snapshots. In total 10 will be attempted.

![Working snapshots](/images/WorkingSnapshots.png)

The snapshot is successful if the number of each commodity is 15. Until your code is correct, you will probably see cases where there is undercounting (commodities < 15) and overcounting ( > 15). Your snapshot code should repeatedly pass all 10 tests.

Therefore, the core of this task is to modify ONLY the code in PITPlayerModel.java. (No other file should be edited.) Modify the player model so that it implements the snapshot algorithm for the PITplayers and pass the results to the PITsnapshot servlet.

### Task – Add monopoly-seeking behavior to the players in a simulation
The goal of the original *Pit* card game is to corner a commodity market by having all 15 cards of that single commodity; for example, all 15 cards of Rhodium.

The simulation currently randomly originates trades. Add logic to the simulation so that each player will attempt to collect a monopoly of one commodity.

In order to do so:  
A. You can only modify PITPlayerModel.java.  
B. The logic must be generalizable and work for:
 - Any number of players
 - Any number of commodities
 - Any commodity names
 - Any number of commodity cards per player  

C.	Your logic can refer to the information in a TenderOffer, including TenderOffer.sourcePlayer, to decide how to respond to an offer.  
D.	Your logic can decide what playerNumber to tender a new offer to (currently new offers are tendered to a random other player). This playerNumber cannot be hard coded, however (e.g. always offer trades to hardcoded player 4).

The intention is not that your solution must achieve a monopoly. Rather, you should have a hypothesis, implement it, and discuss the outcomes you see in the snapshots (see the final task).

Once you have achieved a monopoly, **do not stop trading**. Yes, this will force the Player to give away one of their matching cards and consequently lose their monopoly. This is ok because trading must continue to test all 10 snapshots.

### Task – Use snapshots to understand the behavior of a system
Create a pdf document named MonopolyStrategyAnalysis.pdf with the following content:

 1. Name and Andrew ID  
 1.	Describe the strategy you implemented for having each player work toward a monopoly. Write this in prose so that a non-technical person would understand the approach you took for trading toward a monopoly. (200 words maximum)  
 1.	Looking at your sequences of 10 snapshots, perhaps run several or many times, describe what you find interesting about any patterns of players moving toward achieving monopolies. How well did these patterns match or not match what you expected of your strategy? How well did your strategy result in one more players achieving monopolies? (400 words maximum)

**Grace days:**
You may use your remaining grace days on this project, even though these days go into finals week.

### Use and Borrowing Code
In this project you have been given substantial code to start with.  You do not need to cite this code nor add additional comments to it. You **do**, however, need to comment your code that you add to it.

You should not need to borrow any code from elsewhere, but if you do, be sure to cite it.

**We will use a software similarity detection system to analyze your code in comparison to the submission of others.  Be aware that this looks at the underlying structure of code, not surface features such as comments and variable names. Please review the Policy on collaboration and cheating in the Syllabus.**

### What to turn in
1. Create a directory named with your Andrew ID (and only your Andrew id).
2. Take screen shots of a successful snapshot (because of its length, it will probably take more than one) and put it into this new directory.
3. Copy PITPlayerModel.java (only!) into the directory.  This should have been the only file you modified.	**You should not include your whole project, only PITPlayerModel.java.**
4. Put the file MonopolyStrategyAnalysis.pdf in the directory.
5. Zip the directory containing these files

Therefore your zip file, named YourAndrewID.zip, should contain ONLY:
- A few screenshots
- PITPlayerModel.java
- MonopolyStrategyAnalysis.pdf

6. Submit the zipped file to Canvas.
 (Note: You must have used the correctly named Connection Factory and Queues to get full credit.)
