# Warframe IRC Riven Research
The way rivens are linked throughout the IRC chat is through specialized strings. This string is complicated and messy, however it is easy to read patterns that occur.
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
The way you can decode this string is by first converting it from Base64 to Hexadecimal.
Below is a simple function to do just that, and to convert the hexadecimal back to the base64 string.
```
def b64(string):
	x = base64.b64decode(string).hex()
	return x
def b64d(string):
	x = bytes.fromhex(string)
	y = base64.b64encode(x).decode()
	return y
```
The very first character in the hexadecimal string will tell you how many positives and negatives the riven will have.
|Prefix|Combo|
|--|--|
|F|3+1-|
|D|2+1-|
|E|3+|
|C|2+|

Now onto how to read the stats, this documentation will cover how to read 3+1-, however it should be very easy to apply this to the others.
<br>The hexadecimal for the riven above is `F7 3E BD 5B 97 13 F6 99 41 80 BD 55 7B 47 A3 D6 89 16 3C 6A 6B 4A 00 00`.
<br>The bold portions will be where each stat name is located, the 7 chars after each stat is the stat value.
<br> F**7 3**E BD 5B 97 **13** F6 99 41 8**0 B**D 55 7B 47 **A3** D6 89 16 3C 6A 6B 4A 00 00
<br>Through this, we should get the following result.
<br>Stat 1: **73** EBD5B97
<br>Stat 2: **13** F699418
<br>Stat 3: **0B** D557B47
<br>Stat 4: **A3** D689163

Getting the weapon is a bit more tricky as it can vary across different weapons, the same data that can influence the mastery rank of the riven will also influence the weapon.
<br>F7 3E BD 5B 97 13 F6 99 41 80 BD 55 7B 47 A3 D6 89 16 3**C 6A 6B** 4A 00 00
<br>Weapon: C 6A 6B

Polarity is the following **4**A
<br>Rank is the following 4**A**
<br>The remaining data is how many rolls the riven has.

# Additional Notes
When cross-referencing a bunch of rivens to get the stat names, they will vary and range from weapon type, so it is best to separate them based on that first part of the string, the `OMG-LotusRifleRandomModRare`.
<br>Collecting weapon data from chat will be time consuming, as it is very messy, however you can predict off the data you have based on that string.
<br>I have provided a QoL command below to use through a discord bot for ease of use/reading, however it only works on 3+1- rivens.
```
@commands.command(name='w')
    async def irc_riven(self, ctx, string):
        def b64(string):
            x = base64.b64decode(string).hex()
            output = [x[i:i+2].upper() for i in range(0, len(x), 2)] # groups of 2
            return output
        replace_me = ['[', ']']
        for i in replace_me:
            string = string.replace(i, '')
        string = string.split(':')
        type, riven = string[0], string[1]
        if len(riven) != 32:
            await ctx.send("Only 3+1- are supported.")
            return
        riven = b64(riven)
        stat_1 = f"{riven[0][1]}{riven[1][0]}"
        stat_1v = f"{riven[1][1]}{riven[2]}{riven[3]}{riven[4]}"
        stat_2 = f"{riven[5]}"
        stat_2v = f"{riven[6]}{riven[7]}{riven[8]}{riven[9][0]}"
        stat_3 = f"{riven[9][1]}{riven[10][0]}"
        stat_3v = f"{riven[10][1]}{riven[11]}{riven[12]}{riven[13]}"
        stat_4 = riven[14]
        stat_4v = f"{riven[15]}{riven[16]}{riven[17]}{riven[18][0]}"
        weapon = f"{riven[18][1]} {riven[19]} {riven[20]}"
        polarity = f"**{riven[21][0]}**{riven[21][1]}"
        rank = f"{riven[21][0]}**{riven[21][1]}**"
        rolls = f"**{riven[22]} {riven[23][0]}**{riven[23][1]}"
        string = f"Hex: `{' '.join(riven)}`"
        string+=f"\nType: {type}"
        string+=f"\nStat 1: **{stat_1}** {stat_1v}"
        string+=f"\nStat 2: **{stat_2}** {stat_2v}"
        string+=f"\nStat 3: **{stat_3}** {stat_3v}"
        string+=f"\nStat 4: **{stat_4}** {stat_4v}"
        string+=f"\nWeapon: {weapon}"
        string+=f"\nPolarity: {polarity}"
        string+=f"\nRank: {rank}"
        string+=f"\nRolls: {rolls}"
        await ctx.send(string)
```
