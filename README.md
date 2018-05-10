CPSC353
Text In Image

Scott Tran
Professor: Reza Nikoopour
California University State, Fullerton
Project Description

This program was designed to encode and decode messages on an image. 
The sequence will begin from bottom right to top left.
The first 11 pixel will contain the length of the message stored in binary due to the least significant bits.
The last significant bit for the RGB values will be switch to encode the message that is being stored.
Finally converting the message to binary and binary back into the message.

Text Editor Used: Pluma
OS: UbuntuMate
Language: Python 3.5.2 (Most up-to-date on UbuntuMate)

How to execute

To encode:

1. Write the secret message on code2hide.txt. 
2. Execute:$ python3 main.py -e code2hide.txt testImage.png <image>.png

To decode:

$ python3 main.py -d <image>.png 

Stegenography Project CPSC353

optional arguments:
  man       //Open up help menu
  -d        //decode on an image
  -e        //encode on an image
