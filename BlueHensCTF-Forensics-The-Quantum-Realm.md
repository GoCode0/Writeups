I download the Antman.jpeg file, you can use many steg tools (binwalk, foremost ...) to find that the file contains a text file containing base64 encoded text.
I used this command to store the decoded data to a text file: 
```cat could_this_be_it.txt | base64 -d > b64.txt```
On first look, the data is obviously RGB values of an image. We need the data to be formatted in a certain way in order to use it like in this python script:
```
from PIL import Image
import numpy as np
pixels = [
   [(54, 54, 54), (232, 23, 93), (71, 71, 71), (168, 167, 167)],
   [(204, 82, 122), (54, 54, 54), (168, 167, 167), (232, 23, 93)],
   [(71, 71, 71), (168, 167, 167), (54, 54, 54), (204, 82, 122)],
   [(168, 167, 167), (204, 82, 122), (232, 23, 93), (54, 54, 54)]
]
array = np.array(pixels, dtype=np.uint8)
new_image = Image.fromarray(array)
new_image.save('new.png')
```
I used vim to format the data so I can use it in the python program, if you want to learn vim, this will be a great exercise.
The difficulty of the challenge is finding the correct dimensions of the image, the only information we have is the total number of pixels 22400.

Dividers of 22400:1, 2, 4, 5, 7, 8, 10, 14, 16, 20, 25, 28, 32, 35, 40, 50, 56, 64, 70, 80, 100, 112, 128, 140, 160, 175, 200, 224, 280, 320, 350, 400, 448, 560, 640, 700, 800, 896, 1120, 1400, 1600, 2240, 2800, 3200, 4480, 5600, 11200, 22400.

After many tries and fails, I found out that the correct dimensions were 400x56.
First, use in vim theses exact commands, to remove spaces, replace '\n' with ',' from b64.txt:
```
:%s/ //g
:%s/\n/,/g
```
Then make sure you insert the first '[', after that, we will use a macro to set the number of pixels in each horizontal line, counting the number of pixels by eye is impossible, what we can do is to find a looping sequence using vim motions to automate the task, here is what I came up with (make sure your cursor is on the first chars of the file, before first occurence of the char ')'):
type 'qa' to start recording a macro named 'a' then:
```
400f)a],ENTER
ESCAPE
r[lq
```
the final q at the end stops the macro from recording, then all you need to do is to type this remaining command to repeat the sequence 55 times:
```
55@a
```
Make sure the data is properly formatted, insert it in the python script, the script will create the correct image.
