## Reversing TEW-638APB

I had been reading about reversing and emedded devices' vulnerabilities and I thought why not try it by myself. So to begin with I needed a propriate target. I picked up Trendnet TEW-638APB. So now I had my target and I could start reversing.

I did some background research to learn more about reversing embedded devices. I Found some amazing sources of information and i've linked them down below. The first link is complete guide to reversing a embedded device with both physical and network access. It mentioned the possibility to get routers source code from manufacturer because they use open source projects in their products. Trendet made it really easy to found GPL-licenced codes from their webpage. I downloaded gpl source code for tew-638apb. The router uses goahead web server and i found its source from gpl source package. Next i needed to get the web server binary to do more analysis.

The second link is about reversing devices firmware without physical access. In typical router the firmware contains both filesystem and the image. Filesystem is basically all the binaries and configuration files the router needs. Image is just linux kernel. We are not interested in the image since we are trying to found vulnerabilities in web admin access.

First thing i did with the web server binary was to load it in ida pro free. I didn't get far with that because binary was for MIPS and ida pro free version doesn't support MIPS. I knew from previous projects that there are not that many disassemblers available for free. After a bit of googling i found one stackoverflow question about MIPS disassembler. One answer mentioned radare2 disassembler. 

Radare2 looked like perfect candidate for the job but i soon found out that it was much more difficult to get used to than ida. At first i tried gui for radare2 called bokken. Bokken seemed like great tool but i couln't get it to work. It disassembled normal x86 binaries but i couln't do anything with MIP binaries. After lot of googling i couln't find any working GUIs for radare2 so i decided to try the cli version. Radare2 is not really well documented but it has a gitbook manual and many blog posts and youtube videos helps. The thing with MIPS is that many users use radare2 and ida for reversing x86 binaries and not MIPS binaries. Radare2 has support for MIPS and it's specialities like using register t9 when calling functions. I should maybe write more about radare2 at another time.

After opening the binary with radare2 i couldn't even find the main function. Radare2 is supposed to find the main function but it only took me to binary entry point. From strings i knew that the binary used uclibc. I decided to take a look at uclibc entrypoint to find out where the main function was. In the uclibc source i found the entrypoint in assembly. There was also signature for \__uClibc_main which is called by the MIPS entry point. The signature for that function is __uClibc_main (main, ...) so i knew that one of the arguments for that function is main. 

Time to google MIPS calling convention. MIPS places first four arguments in register and the rest is passed in stack. So the function pointer to main would be passed in register a0. But it is not that simple because MIPS abi defines that all functions calls must go through t9 register and it also uses gloal offset table to access defines symbols.


http://jcjc-dev.com/2016/04/08/reversing-huawei-router-1-find-uart/

http://www.devttys0.com/2011/05/reverse-engineering-firmware-linksys-wag120n/#comment-992174

https://www.cr0.org/paper/mips.elf.external.resolution.txt

http://reverseengineering.stackexchange.com/questions/4040/disassembling-mips-binaries

https://git.uclibc.org/uClibc/tree/libc/sysdeps/linux/mips/crt1.S


