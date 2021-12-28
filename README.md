#!/usr/bin/perl -l
use File::Copy;
# Start Config
$access_log = "/siebelfs/ws-proxy/httpd/logs/access_log";
$default_log = "access"; # access or error
$default_lines = "100";

# End Config
$HOSTNAME = `hostname`;
chomp $HOSTNAME;
$filename= "$HOSTNAME.html";

my $message = <<"END_MSG";
<style>
#logstbl {
    font-family: "Trebuchet MS", Arial, Helvetica, sans-serif;
    border-collapse: collapse;
    width: 70%;
}

#logstbl td, #logstbl th {
    border: 5px solid #ddd;
    padding: 12px;
}

#logstbl tr:nth-child(even){background-color: #f2f2f2;}
#logstbl tr:hover {background-color: #ddd;}

</style>

END_MSG
{$in{'lines'} = $default_lines}
{$in{'log'} = $default_log}
&showlog;


sub showlog{
open(my $fh,'>',$filename);
print $fh "<HTML>";
print $fh "<HEAD>";
print $fh "<TITLE>Viewing $in{'log'} log</TITLE>";
print $fh "</HEAD>";
print $fh "$message";

print $fh "</HTML>";
print $fh "<H1>Viewing webproxy $in{'log'} log last $in{'lines'} lines </H1><TABLE id=logstbl>";
if($in{'log'} eq "access"){
open(LOG, "$access_log");
@line = <LOG>;
close(LOG);
#Reassigning Aarey to remove quit
@line = grep !/quit/, @line;
$num = @line;
for($done=0;$done<$in{'lines'};$done++){
$num--;
($start, $request, $response, $ref, $other, $browser, $end) = split (/\"/, $line[$num]);
($browser, $end) = split (/ \(/, $browser);
($client, $other) = split (/\s/, $line[$num]);
($start, $end) = split (/\- /, $line[$num]);
($user, $bad) = split (/ \-/, $end);
($start, $end) = split (/\[/, $line[$num]);
($time, $line[$num]) = split (/\]/, $end);
if($response =~ /^ 200/){$resp = "OK"}
if($response =~ /^ 500/){$resp = "Server error"}
if($response =~ /^ 404/){$resp = "Page not found"}
if($response =~ /^ 403/){$resp = "Authorisation required"}
if($response =~ /^ 401/){$resp = "Forbidden"}
if($response =~ /^ 400/){$resp = "Bad request"}

print $fh "<TR><TD width=25%><B>At:</B> $time<BR><B>User:</B> $client $user<BR></TD>";
if($resp eq "OK"){
print $fh "<TD width=75%><BR><B>Request:</B> $request<BR><B>Response:</B> $resp ($response)</TD></TR>";
} else {
print $fh "<TD bgcolor=RED width=50%><BR><B>Request:</B> $request<BR><B>Response:</B> $resp ($response)</TD></TR>";
}

$tm=$time;
}
print $fh "</TABLE>";
close $fh;
move ($filename, "/SPS_HC/temp/CRM-SALES_DEV_4.0/$filename");
}
}
exit;




