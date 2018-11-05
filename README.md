# Tips for using Avaya 96xx phones in a 3rd party call control environment

This repository contains a sample 46xxsettings.txt file to connect Avaya 96xx phones to a PBX via a SIP proxy.  It also contains a sample Kamailio SIP proxy configuration file.

My reason for creating this SIP proxy configuration file was originally to assist an Avaya 96xx phone in thinking it was in a proper Avaya environment talking to a PPM Server.  It had additional functionality over the basic "SIPPING 19" mode including a contacts list and speed dials on the extra line buttons.  I ran into one major issue with running the phone in this "AST" mode as they call it.  For some reason, if the phone tries to accept an attended transfer call or tries to complete a conference call, it fails.  Depending on which buttons you press in these failure scenarios, it can also soft lock the phone requiring it to be rebooted.  I eventually abandoned the idea of running my phones in "AST" mode and will be sticking to "SIPPING 19" mode.

When the Avaya 96xx phones are in "SIPPING 19" mode, they don't properly display caller name information.  This proxy configuration file adds the P-Asserted-Identity header to INVITE and INVITE replies that are sent to the phone. This allows the user to see who is calling or who they called (the who they called part only works for 3CX PBX in my configuration file but could be adapted to query any DB).

The 46xxsettings.txt file and avaya-sip-proxy.cfg file both contain comments with some extra information but here are some random things to be aware of and general notes about my own testing:
* Shawn Lockner did a ton of work on reverse engineering how the PPM server works.  He developed Asterisk patches and a PPM Server in PHP.  A thread with all his work and several conversations with me can be found here: [link text itself]https://community.freepbx.org/t/avaya-96x1-extended-features/40543
* Avaya publishes a PPM Interface Specification document that contains samples of most of the PPM requests and responses and a WSDL file that contains the SOAP API definitions that PPM uses.  This can be found on the Avaya Dev Connect website.  I can't remember if you need a login to access these downloads.  If you do I think its free to create one.
* I don't really understand what the `SET CONNECTION REUSE` config parameter is for in 46xxsettings.txt but with its default 1 option enabled, the phone will not work behind a proxy.  Since firmware 7.1, this parameter can only be set to 1, making it impossible to make the phone work behind a proxy.  If you find a way around this, let me know.
* When viewing Avaya's sample 46xxsettings.txt files they release with each firmware update, pay attention to the section after each config parameter where they specify if its an H.323 or SIP setting and which range of phones and firmware versions it applies to.  You'll find that the J1xx series of phones and the Avaya Vantage platforms are the only phones they really intended to be run on 3rd party call control platforms
* My organization migrated from an Avaya Aura 6.3 platform with Communication Manager to 3CX.  We did not have Session Manager or any of the Avaya SES stuff at any point and the phones were running H.323 firmware
* During our migration from Avaya CM to 3CX, I built a SIP trunk between the two so the systems could call between each other with normal 4 digit dialing.  Apparently Avaya CM can use a SIP trunk to route calls but cannot support registration of SIP endpoints.  I guess they rely on Session Manager to do this piece.
* I found when testing the phones with 3CX, they didn't display caller name information.  If you see caller name information on your PBX without any additional configuration, I'd be interested in seeing the SIP dialog and knowing which Avaya firmware you are using.  I'd assume that your PBX always sends P-Asserted-Identity headers even for internal calls.
* My reason for having the phone behind a proxy in the first place is because most PBXs can't easily modify the headers for normal internal calls
* Avaya 96xx phones only send and receive SIP messages over TCP or TCP w/ TLS, I can't tell based on the notes in Avaya's provided 46xxsettings.txt examples if this was always the case
* In firmware 7.1, on 96x1 phones only you can force the phone to login to a specific extension using just the 46xxsettings.txt using `SET FORCE_SIP_USERNAME`, `SET FORCE_SIP_PASSWORD`, and `SET FORCE_SIP_EXTENSION` configuration parameters
