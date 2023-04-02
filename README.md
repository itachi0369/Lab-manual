set ns [new Simulator]
#Open the NS trace file
set tracefile [open prgm1.tr w]
$ns trace-all $tracefile
#Open the NAM trace file
set namfile [open prgm1.nam w]
$ns namtrace-all $namfile
#Create 3 nodes
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
#Createlinks between nodes
$ns duplex-link $n0 $n1 100.0Mb 10ms DropTail
$ns queue-limit $n0 $n1 50
$ns duplex-link $n1 $n2 0.5Mb 10ms DropTail
$ns queue-limit $n1 $n2 15
#Give node position (for NAM)
$ns duplex-link-op $n0 $n1 orient right
$ns duplex-link-op $n1 $n2 orient right
#Setup a UDP connection
set udp0 [new Agent/UDP]
$ns attach-agent $n0 $udp0
set null1 [new Agent/Null]
$ns attach-agent $n2 $null1
$ns connect $udp0 $null1
#Setup a CBR Application over UDP connection
set cbr0 [new Application/Traffic/CBR]
$cbr0 attach-agent $udp0
$cbr0 set packetSize_ 500
$cbr0 set rate_ 1.0Mb
$cbr0 set interval_ 0.005
$cbr0 set random_ null
$ns at 0.0 "$cbr0 start"
$ns at 50.0 "$cbr0 stop"
#Define a 'finish' procedure
proc finish {} {
global ns tracefile namfile
$ns flush-trace
close $tracefile
close $namfile
exec nam prgm1.nam &
exit 0
}
$ns at 50.0 "finish"
$ns run
AwkScript for Counting packet drop:
BEGIN{}
{
if($1 == "d" && $6 > 100){
count++;
}
}
END{
print "No.of Packets dropped are:", count;
}
