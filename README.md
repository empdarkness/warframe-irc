# Preface
The purpose of this document is to even out the playing field on rivens within trade chats of all platforms and possibly get it patched.
The following way to get into IRC has been reported weeks in advance of this document, but there have been no attempts at patching it. ¯\\_(ツ)_/¯
Credits: [rRedLim](https://github.com/rRedLim)
# Vulnerability
What this allows you to do is access all chats within the game in all platforms. You can access squad chat, clan chat, region chat (ew), trade chat.
<br>You can see the account ids of everyone connecting to chat, their masked IP, and platforms.
<br>This allows you to detect name changes if a person were to ever connect to a chat you're watching.

# Getting Into IRC
The steps here may be rough, but generally it works.
<br>The process with the auth token and commands has to be somewhat fast, so the token does not expire.
1. Clear warframe cache files to wipe the "saved" ban from previous attempts.
2. Connect with vpn for safe measure.
3. Make 2 accounts through vpn. (Emails generally are outlook or something popular)
<br>Account 1 will be the "sacrifice" account used to generating an IRC password for Account 2.
4. Do the tutorial so you're in ship on Account 1.
5. Connect a debugger to the client and setup breakpoints on IRC connectivity.
6. You will want the client to reconnect to IRC to trigger the breakpoints, so suspend the process with the program of choice and recover it once enough time has passed.
7. Modify memory at breakpoints to change the account id and nonce to Account 2.
<br>The nonce will need to be the same length, so keep logging out and in until it is the same.
<br>You can see the nonce by trying to buy platinum through the game, it will open a browser link containing your account id and current nonce.
9. Connect to IRC server with modified standalone client such as HexChat to get the Auth token.
10. Use this auth token in the process of editing memory to generate the password.
11. Once you have the password, you now can connect with standalone client.
12. /NICK account2_name
13. /USER account2_ID_0 0 * irc_password
14. You're now connected and have free reign.

# Reading Riven Mods
The way rivens are linked throughout the IRC chat is through specialized strings.
<br>A quick and easy way to obtain one of these strings is to post one to chat and check your EE.log, there should be an outbound message containing the string.

For example: `[OMG-LotusRifleRandomModRare:9z69W5cT9plBgL1Ve0ej1okWPGprSgAA]`
```
Mutalist Cernos Acri-visicron
203.4 Damage
164.4 Critical Damage
172.8 Critical Chance
-41.5 Damage to Corpus
MR13 - 0 Rolls - Vazarin - Rank 8
```
The way you can decode this string is by first converting it from Base64 to Binary.
<br>Below is a simple function to do just that.
<br>The `9z69W5cT9plBgL1Ve0ej1okWPGprSgAA` part of the string is the thing you want to use in the function.
```
def b64(string):
	x = riven.encode('ascii')
	dec = base64.decodebytes(x)
	riven = "".join(["{:08b}".format(x) for x in dec])
	return riven
```
The following data will be at these positions:
```
pos1 = riven[4:9]
pos1_value = riven[9:40]
pos2 = riven[40:45]
pos2_value = riven[45:76]
pos3 = riven[76:81]
pos3_value = riven[81:112]
neg = riven[112:117]
neg_value = riven[117:148]
mr = int(riven[165:170], 2)
polarity = pol.get(riven[170:174])
rolls = int(riven[178:190], 2)
weapon = riven[157:165]
```
New weapons and stats can and will shift existing data, so keep that in mind.

<br>You can "grade" rivens by passing the stat value through this function.
```
def btd(binary):
	dec = int(binary, 2)
	return dec
```
With enough data collection, you can find value ranges for each grade.
