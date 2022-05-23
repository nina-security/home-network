# Brother MFC-L2750DW

## How to Switch the Device Language
My new device came with Italian language setup. Using the language options on the device display, I could only choose between Italian, Spanish, and Portugues.

It turned out that it was a bit tricky to switch to my desired language. You need to provide a 4-digit code consisting of 2 model digits and 2 language digits.
In the first attempt, I selected the wrong model code as I just copy-pasted it from some write-up, which resulted in some functions, such as ADF scanning, not working.
It was quite annoying to debug this as I first did not realize that the malfunctioning might depend on the code I entered.

So here's what finally worked for me:

1) Look up the model/language code for your specific device in the Service Manual, e. g. on [manualslib.com](https://www.manualslib.com/manual/1582425/Brother-Dcp-7090.html#product-MFC-L2750DW). I found the codes for my device in a chapter named *Configure for country/region and model (Function code 74)*. Ensure that you look up the language code for the correct model. In my case it was `0404`, where the first two digits belong to the model and the last two digits to the language (English UK).
1) Switch on the device.
1) Hold the home button for several seconds. 4 lines should appear, of which the last two are empty.
1) Click on the last row.
1) Enter the following sequence to switch into maintenance mode: `*2864`
1) Enter the following to switch into language menu: `74`
1) Confirm with `Mono Start`
1) Enter the model/language code, i. e. in my case `0404`.
1) Confirm with `Mono Start`
1) To exit the menu, enter `99`. The device will restart.

Note for myself: The initial code was `0415`.

## Client Software
If you want to use your laptop/computer for scanning, you need to install respective software. 

For my model, the software downloads are provided here: 
[Software Downloads for MFC-L2750DW](https://support.brother.com/g/b/downloadlist.aspx?c=as_ot&lang=en&prod=mfcl2750dw_us_eu_as&os=10013)

On my Windows10 laptop, I installed the software with the following titles in this order:

- Full Driver & Software Package (Recommended)
- ControlCenter4 Update Tool 