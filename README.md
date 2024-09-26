java c Multi-threading - Token Bucket Emulation in C 
You will emulate/simulate a traffic shaper that transmits/services packets controlled by a token 
bucket filter depicted below using multi-threading within a single process. If you are not 
familiar with pthreads, you should read the textbook. 
 Multi-threading - Token Bucket Emulation in C 
 
 
Figure 1 above depicts the system you are required to emulate. The token bucket has a capacity 
(bucket depth) of B tokens. Tokens arrive into the token bucket according to an unusual arrival 
process where the inter-token-arrival time between two consecutive tokens is 1/r. We will 
call r the token arrival rate (although technically speaking, it's not exactly the token arrival 
rate; please understand that this is quite different from saying that the tokens arrive at a constant 
rate of r). Extra tokens (overflow) would simply disappear if the token bucket is full. A token 
bucket, together with its control mechanism, is referred to as a token bucket filter. 
 
Packets arrive at the token bucket filter according to an unusual arrival process where the interarrival
time between two consecutive packets is 1/lambda. We will call lambda the packet 
arrival rate (although technically speaking, it's not exactly the packet arrival rate; please 
understand that this is quite different from saying that the packets arrive at a constant rate 
of lambda). Each packet requires P tokens in order for it to be eligiable for transmission. (Packets 
that are eligiable for transmission are queued at the Q2 facility.) 
When a packet arrives, if Q1 is not empty, the packet will just get queued onto the Q1 facility 
since packets must be processed in the first-come-first-serverd order. (Please note that, in this 
case, you do not have to check if there is enough tokens in the bucket so you can move the 
packet at the head of Q1 into Q2 and you need to understand why you do not need to perform 
such a check.) Otherwise (i.e., Q1 is empty), you must check to see if the token bucket has P or more tokens in 
it. If the token bucket has P or more tokens in it, you must remove P tokens from the token 
bucket and move the packet into Q2 (although technically speaking, you are required to first add 
the packet to Q1 and timestamp the packet, remove the P tokents from the token bucket and the 
packet from Q1 and timestamp the packet, before moving the packet into Q2 and get another 
timestamp for the packet), and wake up the servers in case they are sleeping. On the other hand, 
if the token bucket does not have enough tokens, the packet gets queued into the Q1 facility. 
Finally, if the number of tokens required by a packet is larger than the bucket depth, the packet 
must be dropped and not get added to Q1 (otherwise, it will block all other packets that follow 
it). 
 
The transmission facility (denoted as S1 and S2 in the above figure and they are referred to as the 
"servers") serves packets in Q2 in the first-come-first-served order and at a transmission/service 
rate of mu per second. When a server becomes available, it will dequeue the first packet 
from Q2 and start transmitting/servicing the packet. When a packet has received 1/mu seconds of 
service, the packet leaves the system. You are required to keep the servers as busy as possible. 
 
When a token arrives at the token bucket, it will add a token into the token bucket. If the 
bucket is already full, the token will be lost. It will then check to see if Q1 is empty. If Q1 is not 
empty, it will see if there is enough tokens to make the packet at the head of Q1 be eligiable for 
transmission (packets in Q1 must be served in the first-come-first-served order). If it does, it will 
remove the corresponding number of tokens from the token bucket, remove that packet 
from Q1 and move it into Q2, and wake up the servers in case they are sleeping. Please note that 
under this scenario, the token bucket must be empty at this time and there would be no need to 
check to see if the packet now at the head of Q1 is eligible for transmission or not. 
Technically speaking, the "servers" are not part of the "token bucket filter". Nevertheless, it's part 
of this assignment to emulation/simulation the severs because the servers are considered part of 
the "system" to be emulated. (For the purpose of this spec, we will use the term "emulation" and 
"simulation" interchangeably.) 
 
Our system can run in only one of two modes. 
Deterministic : In this mode, all packet inter-arrival times are equal to 1/lambda seconds and all 
transmission/service times are equal to 1/mu seconds (both these values must 
be rounded to the nearest millisecond), and all packets require 
exactly P tokens. If 1/lambda is greater than 10 seconds, please use an interarrival
 time of 10 seconds. If 1/mu is greater than 10 seconds, please use a 
transmission/service time of 10 seconds. 
 
Trace-driven : In this mode, we will drive the emulation using a trace specification file (will be 
referred to as a "tsfile"). Each line in the trace file specifies the inter-arrival 
time of a packet, the number of tokens it need in order for it to be eligiable for 
transmission, and its transmission/service time. (Please note that in this mode, it's perfectly fine if an inter-arrival time or a transmission/service time is greater 
than 10 seconds.) If you are running in the trace-drive mode, you 
must not validate or read the entire tsfile before you start your simulation 
because that would delay the start of simulation. 
 
In either mode, if 1/r is greater than 10 seconds, please use an inter-token-arrival time of 10 
seconds. Otherwise, please round the inter-token-arrival time to the nearest millisecond. 
Your job is to emulate the packet and token arrivals, the operation of the token bucket filter, the 
first-come-first-served queues Q1 and Q2, and servers S1 and S2. You also must produce a trace 
of your emulation for every important event occurred in your emulation. Please see more 
details below for the requirements. 
You must use: 
• one thread for packet arrival 
• one thread for token arrival 
• one thread for each server 
You must not use one thread for each packet. 
In addition, you must use at least one mutex to protect Q1, Q2, and the token bucket. (It is 
recommended that you use exactly one mutex to protect Q1, Q2, and the token bucket.) 
Finally, Q1 and Q2 must have infinite capacity (i.e., you should use My402List from previous 
assignment to implement them and not use arrays). 
 
 
Please use a Makefile so that when the grader simply enters: 
 make warmup2 
an executable named warmup2 is created (minor variation is permitted if you document it). 
 
 
Commandline 
 
The command line syntax (also known as "usage information") for warmup2 is as follows: 
 
 warmup2 [-lambda lambda] [-mu mu] [-r r] [-B B] [-P P] [-n num] [-t 
tsfile] 
 
Square bracketed items are optional. You must follow the UNIX convention that commandline 
options can come in any order. (Note: a commandline option is a commandline argument that 
begins with a - character in a commandline syntax specification.) Unless otherwise specified, 
output of your program must go to stdout and error messages must go to stderr. 
The lambda, mu, r, B, and P parameters all have obvious meanings (according to the description 
above). The -n option specifies the total number of packets to arrive. If the -t option is specified, tsfile is a trace specification file that you should use to drive your emulation. In this 
case, you should ignore the -lambda, -mu, -P, and -n commandline options and run your 
emulation in the trace-driven mode. You may assume that tsfile conforms to the tracefile 
format specification. (This means that if you detect an error in this file, you may simply print an 
error message and call exit() to quit your program, even if you have started your simulation. 
You must not try to perform error recovery if the input file does not conform to the tracefile 
format specification.) If the -t option is not specified in the commandline, you must run your 
emulation in the deterministic mode. 
The default value (i.e., if it's not specified in a commandline option) for lambda is 1 (packets per 
second), the default value for mu is 0.35 (packets per second), the default value for r is 1.5 
(tokens per second), the default value for B is 10 (tokens), the default value for P is 3 (tokens), 
and the default value for num is 20 (packets). B, P, and num must be positive integers (i.e., > 0) 
with a maximum value of 2147483647 (0x7fffffff). lambda, mu, and r must be positive real 
numbers (i.e., > 0). 
 
 
Running The Code and Program Output 
 
The emulation should go as follows. At emulation time 0, all 4 threads (arrival thread, token 
depositing thread, and servers S1 and S2 threads) got started. The arrival thread would sleep so 
that it can wake up at a time in an attempt to match the inter-arrival time of the first packet (i.e., 
either according to lambda or line 2 of a tracefile). At the same time, the token depositing thread 
would sleep so that it can wake up at a time in an attempt to match the inter-token-arrival time 
between consecutive tokens (i.e., 1/r seconds) and would try to deposit one token into the token 
bucket each time it wakes up. The actual arrival time of the first packet p1 is denoted as time T1, 
the actual arrival time of the 2nd packet p2 is denoted as time T2, and so on. 
As a packet or a token arrives, or as a server becomes free, you need to follow the operational 
rules of the token bucket filter. Since we have four threads accessing shared data structures, you 
must use the tricks you learned from Chapter 2 related lectures. Please also check out the slides 
for this assignment for the skeleton code for these threads. 
You are required to produce a detailed trace for every important event occurred during the 
emulation and every such event must be timestamped. Each line in the trace must correspond to 
one of the following situations: 
• If a packet is served by a server (server S1 is assumed below for illustration), 
there must be exactly 7 output lines that correspond to important events related to this 
packet. They are (please see the operational rules of the token bucket filter regarding the 
meaning of these events): 
• p1 arrives, needs 3 tokens, inter-arrival time = 503.112ms 
• p1 enters Q1 
• p1 leaves Q1, time in Q1 = 247.810ms, token bucket now has 0 token 
• p1 enters Q2 
• p1 leaves Q2, time in Q2 = 0.216ms • p1 begins service at S1, requesting 2850ms of service 
 p1 departs from S1, service time = 2859.911ms, time in system = 
3109.731ms 
Please note the following: 
o The value printed for "inter-arrival time" must equal to the timestamp of the 
"p1 arrives" event minus the timestamp of the "arrives" event for the previous 
packet. (You can assume that packet p0 arrived at time 0, even though there is no 
packet p0.) 
o The value printed for "time in Q1" must equal to the timestamp of the "p1 
leaves Q1" event minus the timestamp of the "p1 enters Q1" event. 
o The value printed for "time in Q2" must equal to the timestamp of the "p1 
leaves Q2" event minus the timestamp of the "p1 enters Q2" event. 
o The value printed for "requesting ???ms of service" must be the requested 
service time (which must be an integer) of the corresponding packet. 
o The value printed for "service time" must equal to the timestamp of the "p1 
departs from S1" event minus the timestamp of the "p1 begins service at 
S1" event (and it should be larger than the requested service time printed for the 
"begin service" event). 
o The value printed for "time in system" must equal to the timestamp of the "p1 
departs from S1" event minus the timestamp of the "p1 arrives" event. 
• If a packet is dropped, you must print: 
 p1 arrives, needs 3 tokens, inter-arrival time = 503.112ms, dropped 
Please note that the value printed for "inter-arrival time" must equal to the timestamp of 
the "p1 arrives" event minus the timestamp of the "arrives" event for the previous packet. 
• If  is pressed by the user, you must print the following (and print a '\n' before it 
to make sure that it lines up with all the other trace printouts): 
 SIGINT caught, no new packets or tokens will be allowed 
Please understand that in order for the above to get printed correctly in a trace printout, 
using a signal handler to catch signals may not work. You are strongly advised to use a 
separate SIGINT-catching thread and uses sigwait() (which is covered in Chapter 2). 
• If a packet is removed when it's in Q# (Q1 or Q2) because  is pressed by the 
user, you must print: 
 p1 removed from Q# 
• If a token is accepted, you must print: 
 token t1 arrives, token bucket now has 1 token • If a token is dropped, you must print: 
 token t1 arrives, dropped 
• When you are ready to start your emulation, you must print: 
 emulation begins 
• When you are ready to end your emulation, you must print: 
 emulation ends 
 
All the numeric values above are made up. You must replace them with the actual packet 
number, actual number of tokens required, actual server number, measured inter-arrival time, 
measured time spent in Q1, actual number of tokens left behind when a packet is moved into Q2, 
measured time spent in Q2, measured service time, and measured time in the system. 
 
The output format of your program must satisfy the following requirements. 
• You must first print all the emulation paramters. Please see the sample printout for what 
the output must look like. 
• Whenever a token arrives, you must assign a token number to it, and add it to the token 
bucket. You must then print its arrival time, the fact that it has arrived, and the number of 
tokens in the the token bucket. Please see the sample printout for what the output must 
look like. 
• Whenever a packet arrives, you must assign a packet number to it. You must then print its 
arrival time, the fact that it has arrived, the number of tokens it needs for transmission, 
and the time between its arrival time and the arrival time of the previous packet. Please 
see the sample printout for what the output must look like. 
You then must append this packet onto Q1. Afterwards, you must then print the time this 
packet entered Q1 and the fact that it has entered Q1. Please see the sample printout for 
what the output must look like. 
Later on, when this packet leaves Q1, it removes the correct number of tokens from the 
token bucket. You must then print the time this packet leaves Q1, the fact that it has 
left Q1, the amount of time it spent in Q1, and the number of tokens in the the token 
bucket. Please see the sample printout for what the output must look like. 
You must then append this packet onto Q2. Afterwards, you must then print the time this 
packet entered Q2 and the fact that it has entered Q2. Please see the sample printout for 
what the output must look like. 
Later on, when this packet leaves Q2 and enters the server, you must then print which 
server the packet entered, the time the packet begin service, the fact that it has begun service, and the amount of time it spent in Q2. Please see the sample printout for what the 
output must look like. 
• When emulation ends, you must print all the necessary statistics. Please see the sample 
printout for what the output must look like. If a particular statistics is not applicable (e.g., 
will cause divide-by-zero error), instead of printing a numeric value, please print "N/A" 
followed by an explanation (such as, for example, "no pa代 写program、C/C++
代做程序编程语言cket was served"). Please note 
that your program output must never contain any "NaN" (which means "not-a-number"). 
Sample Printout: Here is what your program output must look like (values used here are just a 
bunch of unrelated random numbers for illustration purposes): 
 
Emulation Parameters: 
 number to arrive = 20 
 lambda = 2 (print this line only if -t is not specified) 
 mu = 0.35 (print this line only if -t is not specified) 
 r = 4 
 B = 10 
 P = 3 (print this line only if -t is not specified) 
 tsfile = FILENAME (print this line only if -t is specified) 
 
 00000000.000ms: emulation begins 
 00000250.726ms: token t1 arrives, token bucket now has 1 token 
 00000501.031ms: token t2 arrives, token bucket now has 2 tokens 
 00000503.112ms: p1 arrives, needs 3 tokens, inter-arrival time = 503.112ms 
 00000503.376ms: p1 enters Q1 
 00000751.148ms: token t3 arrives, token bucket now has 3 tokens 
 00000751.186ms: p1 leaves Q1, time in Q1 = 247.810ms, token bucket now has 0 token 
 00000752.716ms: p1 enters Q2 
 00000752.932ms: p1 leaves Q2, time in Q2 = 0.216ms 
 00000752.982ms: p1 begins service at S1, requesting 2850ms of service 
 00001004.271ms: p2 arrives, needs 3 tokens, inter-arrival time = 501.159ms 
 00001004.526ms: p2 enters Q1 
 00001005.615ms: token t4 arrives, token bucket now has 1 token 
 00001256.259ms: token t5 arrives, token bucket now has 2 tokens 
 00001505.986ms: p3 arrives, needs 3 tokens, inter-arrival time = 501.715ms 
 00001506.713ms: p3 enters Q1 
 00001507.552ms: token t6 arrives, token bucket now has 3 tokens 
 00001508.281ms: p2 leaves Q1, time in Q1 = 503.755ms, token bucket now has 0 token 
 00001508.761ms: p2 enters Q2 
 00001508.874ms: p2 leaves Q2, time in Q2 = 0.113ms 
 00001508.895ms: p2 begins service at S2, requesting 1900ms of service 
 ... 
 00003427.557ms: p2 departs from S2, service time = 1918.662ms, time in system = 2423.286ms 
 00003612.843ms: p1 departs from S1, service time = 2859.861ms, time in system = 3109.731ms 
 ... 
 ????????.???ms: p20 departs from S?, service time = ???.???ms, time in system = ???.???ms 
 ????????.???ms: emulation ends 
 
 Statistics: 
 
 average packet inter-arrival time =  
 average packet service time =  
 
 average number of packets in Q1 =  
 average number of packets in Q2 =  
 average number of packets at S1 =  
 average number of packets at S2 =  
 
 average time a packet spent in system =  
 standard deviation for time spent in system =  
 
 token drop probability =  
 packet drop probability =  In the Emulation Parameters section, please print the emulation parameters specified by the user 
or the default values mentioned above. Please do not print the "adjusted" values because certain 
parameters are too small. (For example, if lambda is 0.01, you must print 0.01 and not 0.1.) 
 
After Emulation Parameters section comes the Event Trace section. The first column there 
contains timestamps and they correspond to event times, measured relative to the start of the 
emulation. Every emulation event must be timestampted. You need to figure out how to make 
sure that the timestamp values look reasonable (e.g., never decrease in value). Please use exactly 
8 digits (zero-padded) to the left of the decimal point and exactly 3 digits after the decimal 
point (zero-padded) for all the timestamps in this column. All time intervals in the trace 
printout must be printed in milliseconds with exactly 3 digits after the decimal point (zeropadded).
Please remember that a time interval in the trace printout is the difference between two 
timestamps and timestamps are considered integers with a resolution of microseconds. 
Therefore, you must print all the digits. If your code represent timestamps as double, you need to 
make sure that you satisfy this requirement or you may end up losing a lot of points. In the 
printout, after emulation parameters, all values reported must be measured values. 
 
In the Statistics section, the average number of packets at a facility can be obtained by adding 
up all the time spent at that facility (for all relevant packets) divided by the total emulation time. 
The time spent in system for a packet is the difference between the time the packet departed 
from the server and the time that packet arrived. The token drop probability is the total number 
of tokens dropped because the token bucket was full divided by the total number of tokens that 
was produced by the token depositing thread. The packet drop probability is the total number 
of packets dropped because the number of tokens required is larger than the bucket depth divided 
by the total number of packets that was produced by the arrival thread. 
All real values in the Emulation Parameters and Statistics sections must be printed with at least 6 
significant digits. (If you are using printf(), you can use "%.6g". Please note that with 
"%.6g", printf() would omit trailing zeroes.) A timestamp in the beginning of a line of trace 
output must be in milliseconds with exactly 8 digits (zero-padded) before the decimal point and 
exactly 3 digits (zero-padded) after the decimal point. Please note that the timestamps must 
lined up perfectly and have microsecond resolution and the grader needs to see all these 
digits. 
Please use sample means when you calculated the averages. If n is the number of sample, this 
mean that you should divide things by n (and not n-1). 
The unit for time related statistics must be in seconds (and not milliseconds). 
Let X be something you measure. The standard deviation of X is the square root of the variance 
of X. The variance of X is the average of the square of X minus the square of the average of X. 
Please note that we must use the "population variance" (and not a "sample variance") in our 
calculation since we have all the data points. Let E(X) denote the average of X, you can write: 
Var(X) = E(X2
) - [E(X)]2
 When you are keep statistics, you should keep a running average. 
Please note that it's very important that the event time in the printout is monotonically 
increasing (as shown in the sample printout below). This can be difficult to achieve when we 
have multiple threads running in parallel. But since we are using only one mutex, you can use the 
following simple (although not super-efficient) trick. When you are getting the time for an event, 
you must have the mutex locked, and you must not release the mutex until you have printed the 
line of printout that corresponds to that event, i.e., reading the clock and printing out the event is 
done in one atomic operation. 
If the user presses  on the keyboard, you must stop the arrival thread and the token 
depositing thread, remove all packets in Q1 and Q2, let your server finish transmitting/servicing 
the current packet, and output statistics. (Please note that it may not be possible to remove all 
packets in Q1 at the instance of signal delivery. The idea here is that once signal delivery has 
occurred, the only packet you should serve are the ones currently being transmitted/serviced. All 
other packets should be removed from the system.) 
You can divide the packets into 3 categories. 
1. Completed packets: these are the packets that made it all the way to the server and 
completed service at the server. 
2. Dropped packets: these are the packets arrived into the system but never made it even to 
Q1 because it needs too many tokens. 
3. Removed packets: these are the packets that got into Q1 to begin with but never made it 
to the server. 
All packets should participate in the calculation of the average packet inter-arrival time and 
packet drop probability statistics. Only completed packets should participate in the calculation of 
the average packet service time statistics. Only completed packets should participate in the 
calculation of the average number of packets in Q1/Q2/S1/S2 and time spent in system statistics. 
Finally, when no more packet can arrive into the system, you must stop the arrival thread as soon 
as possible. Also, when Q1 is empty and no future packet can arrive into Q1, you must stop the 
token depositing thread as soon as possible. 
Trace Specification File Format 
 
The trace specification file is an ASCII file containing n+1 lines (each line is terminated with a 
"\n") where n is the total number of packets to arrive. Line 1 of the file contains a positive integer 
which corresponds to the value of n. Line k of the file contains the inter-arrival time in 
milliseconds (a positive integer), the number of tokens required (a positive integer), and service 
time in milliseconds (a positive integer) for packet k-1. The 3 fields are separated by space or tab 
characters (or any combination of any number of these characters). There must be no leading or 
trailing space or tab characters in a line. If a line is longer than 1,024 characters (including the 
'\n' at the end of a line), it is considered an error. A sample tsfile for n=3 packets is provided. 
It's content is listed below:  3 (This is the sample tsfile) 
 2716 2 9253 
 7721 1 15149 
 972 3 2614 
In the above example, packet 1 is to arrive 2716ms after emulation starts, it needs 2 tokens to be 
eligible for transmission, and its service time should be 9253ms; the inter-arrival time between 
packet 2 and 1 is to be 7721ms, it needs 1 token to be eligible for transmission, and its service 
time should be 15149ms; the inter-arrival time between packet 3 and 2 is to be 972ms, it needs 3 
token to be eligible for transmission, and its service time should be 2614ms. 
In the above example, you should treat these numeric values as "targets" or your emulation. In 
your trace output, you need to print what you measured (i.e., by reading the clock). It should be 
very unlikely that a measured inter-arrival time or a measured service time has exactaly the 
same value as its corresponding target value. For example, the inter-arrival time of packet 3 is 
suppose to be 972 milliseconds. If the reported actual inter-arrival time between packets 2 and 3 
is exactly 972.000 milliseconds, you should look for bugs in your code! Actually, you should 
probably get a different value every time your rerun your emulation. 
This file is expected to be error-free. (This means that if you detect a real error in a tsfile, you 
must simply print an error message and call exit() immediately. You MUST NOT print 
statistics or attempt to recover from error in this case.) 
You are expected to create your own tsfile to test your program. Make sure you know how to 
create test cases where you know for sure that packets will be wait in Q1, in Q2, or both. You 
should be able to look at your tsfile and predict what will happen in the trace and verify that 
your program printout is consistent with your prediction. 
Minimum Emulation Time 
 
If you have the fastest machine in the universe that there is no overhead anywhere (i.e., 
bookkeeping time is zero everywhere, takes zero time to execute any code, etc.) and it's running 
a real-time OS that always sleeps exactly the amount of time you ask it to sleep, what would be 
the minimum simulation time when you run warmup2? Of course, this depends on the 
parameters of your simulation. Let's take the sample tsfile shown above and think about when 
each packet will leave the simulation if we simply run: 
 ./warmup2 -t tsfile.txt 
1. If there is no overhead anywhere, packet p1 would arrive at exactly 2716ms into the 
simulation. At that time, the token bucket should have more than enough tokens 
for p1 and p1 would start transmitting immediately. Since the transmission time of p1 is 
9253ms, p1 should finish transmission at time 11969ms. 
2. Packet p2 would arrive at exactly 7721ms after the arrival time of packet p1. This means 
that packet p2 would arrive at time 10437ms. At that time, the token bucket should have 
more than enough tokens for p2 and p2 would start transmitting immediately. Since the 
transmission time of p2 is 15149ms, p2 should finish transmission at time 25586ms. 
3. Packet p3 would arrive at exactly 972ms after the arrival time of packet p2. This means 
that packet p3 would arrive at time 11409ms. At that time, the token bucket should have more than enough tokens for p3. But, both servers are busy. Therefore, p3 must wait in 
Q2. The server that transmitted p1 would finish first at time 11969ms and it would start 
transmitting p3 as soon as it becomes available. Since the transmission time of p3 is 
2614ms, p3 should finish transmission at time 14583ms. 
From the above analysis, simulation will end when p2 is transmitted at time 25586ms. By doing 
analysis like this, you can figure out the minimum simulation time of your program. If your 
program runs faster than that, you would know for sure that you have a bug in your code! (Of 
course, if the number of packets is large in an input file, it may not be feasible to do this type of 
analysis by hand.) 
 
 
The grading guidelines and grading data (in w2data.tar.gz) have been made available. 
Please run the scripts in the guidelines on a standard 32-bit Ubuntu 16.04 system. 
 
The grading guidelines is the only grading procedure we will use to grade your program. No 
other grading procedure will be used. Please note that the grader may use a different set of trace 
files and commandline arguments when grading your submission. (We may make minor 
changes if we discover bugs in the script or things that we forgot to test.) It is strongly 
recommended that you run your code through the scripts in the grading guidelines. 
For your convenience, a copy of the grading scripts are made available here: 
 section-A.csh 
 section-B.sh 
 section-A-all.sh 
 section-B-all.sh 
(Technically speaking, the scripts above are not "grading scripts" since they are just scripts 
provided for your convenience to save you some typing. The grader will not run these scripts 
when grading.) 
After you have download the above shell scripts, please put them in the same directory 
as warmup2 and run the following command: 
 chmod 755 section-*.sh section-*.csh 
 
Please also follow the beginning part of the grading guidelines to unpack the grading data file 
(i.e., w2data.tar.gz) so that the script can work properly. By the way, please do not run these 
grading scripts in a Ubuntu shared folder. For unknown reasons, running the grading scripts from 
within a shared folder may not work correctly! 
A Perl script, "analyze-trace.txt", is made available to help you to debug your program printout. 
(I have to name it as if it's a text file. Otherwise, my web server would try to execute the Perl 
script.) Please read the comment at the top of the code to see how to use it. This code only works 
if your printout is in the right format and each regular packet has 7 lines of printout (with their 
timestamps in chronological order) and each dropped packet has 1 line of printout. If your printout has missing lines in the printout or if the lines are in the wrong order, you should fix 
your code and rerun this script! 
For this assignments, please always use pthread_cond_broadcast() to wake up server threads. 
Please do not use pthread_cond_signal() anywhere in your code. (Yes, this is not 
the most efficient way. But since the grader must follow the grading guidelines when grading, 
this would most likely get you the most number of points.) 
In section (A) of the grading guidelines, each test has a minimum emulation time. The numbers 
were obtained using the Minimum Emulation Time analysis mentioned above. If the ru         
加QQ：99515681  WX：codinghelp
