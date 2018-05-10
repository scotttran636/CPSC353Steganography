from PIL import Image
import sys
import math

def str2bin(msg):
    return ''.join(format(ord(x), '08b') for x in msg)

    #Convert the strings of the message into binary in order to be able to manipulate the pixels in the encode function

def bin2str(binary):
    intbin = int(binary, 2)
    msg = intbin.to_bytes((intbin.bit_length() + 7) // 8, 'big').decode()
    return msg
    
    #Convert the binary given in the parameter back into strings using the decode function and returning the message

def encode(img, msg):
    #Encode message inside the lbit of each RGB value inside an image.
    
    imginfo = list(img.getdata())
    newimage = []
    msgbin = str2bin(msg)
    msg_length = len(msgbin)

    msglenbin = format(msg_length, '032b') #Takes the length of the message and converts it into binary and stores that in the first 11 pixels
    
    x, y = img.size #get size of the image and store in x and y
    width = x #set width to the x value of the size
    index = 0
    y = y - 1

    #Get RGB value from each pixel and change the least significant bit
    while msglenbin:
        x = x - 1
        r, g, b = img.getpixel((x, y))
        index -= 1 #pixel position

        if msglenbin:
            r, msglenbin = newrgb(r, msglenbin)
            if not msglenbin:
                newimage.append((r, g, b))
                break

            g, msglenbin = newrgb(g, msglenbin)
            if not msglenbin:
                newimage.append((r, g, b))
                break

            b, msglenbin = newrgb(b, msglenbin)
            if not msglenbin:
                newimage.append((r, g, b))
                break

        newimage.append((r, g, b))

    #Hide the message length in the first 11 pixels of the image starting from bottom right to left top
    while msgbin:
        x = x - 1

        r, g, b = img.getpixel((x, y))
        index = index - 1 #pixel position

        if msgbin:
            r, msgbin = newrgb(r, msgbin)
            if not msgbin:
                newimage.append((r, g, b))
                break

            g, msgbin = newrgb(g, msgbin)
            if not msgbin:
                newimage.append((r, g, b))
                break

            b, msgbin = newrgb(b, msgbin)
            if not msg_length:
                newimage.append((r, g, b))
                break

        newimage.append((r, g, b))

        if x == 0:
            y = y - 1
            x = width

    newimage = newimage[::-1] #Reverse the list in order to put it from left to right order for decode

    newimage = imginfo[:index] + newimage #Copy over rest of pixels for full image
    return newimage

def decode(img):
    #Decode the message hidden inside the image assuming that the message and length has been stored properly.
    #pix = img.load()
    x, y = img.size #get size of the image and store in x and y

    width = x #set width to the x value of the size

    #Reads the first 11 pixel in order to find the message length in binary.
    bitnum = []
    y = y - 1
    for _ in range(11):
        x = x - 1
        r, g, b = img.getpixel((x, y))

        #Finds the least significant bit of the RGB value in order to construct the total number of bits into bitnum.
        tempr = '{0:08b}'.format(r)
        bitnum.append(tempr[-1])

        tempg = '{0:08b}'.format(g)
        bitnum.append(tempg[-1])

        tempb = '{0:08b}'.format(b)
        bitnum.append(tempb[-1])
        
        #if the picture image has less than 11 columns of pixels it will move up a row in order to find the length of message in the first 11 pixels than decodes the message after.
        if x == 0:
            y = y - 1
            x = width

    bitnum = bitnum[:-1] #Extra least significant was discovered and must be removed since only 32 bits are being used

    msg_length = int(''.join(bitnum), 2) #combine bitnum(s) in order to convert to discover binary and length for the message length.

    #Using message length discovered from the 11 pixels in order to determine the message using bin2str function to decode message.
    msg = []
    while len(msg) < msg_length:
        x = x - 1
        #print("X ", x, "Y ", y)
        r, g, b = img.getpixel((x, y))

        tempr = '{0:08b}'.format(r)
        msg.append(tempr[-1])
        if len(msg) == msg_length:
            break

        tempg = '{0:08b}'.format(g)
        msg.append(tempg[-1])
        if len(msg) == msg_length:
            break

        tempb = '{0:08b}'.format(b)
        msg.append(tempb[-1])
        if len(msg) == msg_length:
            break

        #if the picture image has less than 11 columns of pixels it will move up a row in order to find the length of message in the first 11 pixels than decodes the message after.
        if x == 0:
            y = y - 1
            x = width

    msg = bin(int(''.join(msg), 2)) #convert the least significant bits added together into a real binary
    message = bin2str(msg) #binary to string conversion to discover message
    return message


def newrgb(color, binary):
    #Changing the RGB value of a pixel in order to hide message in bits of RGB value
    temprgb = list('{0:08b}'.format(color))
    binary = list(binary)
    temprgb[-1] = binary[0]
    temprgb = ''.join(temprgb)
    color = int(temprgb, 2)
    binary = binary[1:]
    return color, binary


def main():
    #Setting up arguments in order to run program by determining specific argument inputs with the exception of "python3" which will not be counted in the argument count
    if len(sys.argv) == 1:
        print("missing parameter")
    elif len(sys.argv) == 2 and (sys.argv[1] == "man"):
        help_menu()
    elif len(sys.argv) == 3 and (sys.argv[1] == '-d'):
        inImage = sys.argv[2]
        if inImage[-4:] == ".png":
            img = Image.open(inImage)
            msg = decode(img)
            print("Your hidden message is\n", msg)
        else:
            print("Improper parameter input. Check with python3 main.py man. Bad Decode")
    elif len(sys.argv) == 5 and (sys.argv[1] == '-e'):
        infile = sys.argv[2]
        inImage = sys.argv[3]
        outImage = sys.argv[4]
        if (inImage[-4:] == ".jpg" or inImage[-4:] == ".png") and infile[-4:] == ".txt":
            if outImage[-4:] == ".png":
                outImage = outImage[:-4]
            outImage = outImage + ".png"
            file = open(infile)
            msg = file.read()
            file.close()
            img = Image.open(inImage)
            newimg = encode(img, msg)
            img.putdata(newimg)
            img.save(outImage, "PNG")
        else:
            print("Improper parameter input. Check with python3 main.py man. Bad Encode.")
    else:
        print("Improper parameter input. Check with python3 main.py man. Just Bad.")

def help_menu():
    print("Help: python3 main.py man\n")
    print("python3 main.py [-d] [(image).png]\n")
    print("python3 main.py [-e] [input-file] [(image).png/.jpg] [(output image).png]\n")
    print("\nDescription\n")
    print("\tmain.py is used to encode or decode messages within an image file.\n")
    print("\nCommands\n")
    print("\tman --help\tdisplays manual page for instructions on how to run program.\n")
    print("\t[-d] --decode\ttakes <*.png> image and displays the decoded message.\n")
    print("\t[-e] --encode\ttakes <*.jpg> or <*.png> and <*.txt> file as input and takes name for output image<*.png>.\n")


if __name__ == '__main__':
    main()
