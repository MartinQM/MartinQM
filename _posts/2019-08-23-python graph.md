---
layout: post
title: 用Python制作大数量组图
---

最近登陆加拿大一周年了，突然想制作一个100张左右的大数据组图，挑选一些照片，拼成一张照片，还可以发个朋友圈。留作纪念

网上找了找似乎没有什么合适的工具，最后还是决定用python。大致搜了搜根据两位博主的代码修改了一番，链接如下：

这个是主要用来把照片裁剪然后拼接的：

https://www.airbird.info/2018/sw-python-pillow-pintu/

这个是用来压缩图片的，有时候图片太大不好处理时间也长：

https://zhuanlan.zhihu.com/p/32246003

我自己是在windows环境，Anaconda下用jupyter notebook跑的。安装一个pillow库即可。

具体我自己略做修改的代码如下：
主要更改是加了一个根据数字排序，这样就可以做出来按时间顺序的组图了。
压缩的时候建两个文件夹`input` `output`然后把照片和压缩代码放在`input`文件夹运行就可以了。
需要注意只能处理.jpg 后缀的文件，.JPG后缀的文件需要改一下。我没有几张.JPG的就手动改了。

```
from glob import glob
from PIL import Image
import os
def resize_images(source_dir, target_dir, threshold):
    filenames = glob('{}/*'.format(source_dir))
    if not os.path.exists(target_dir):
        os.makedirs(target_dir)
    for filename in filenames:
        filesize = os.path.getsize(filename)
        if filesize >= threshold:
            print(filename)
            with Image.open(filename) as im:
                width, height = im.size
                if width >= height:
                    new_width = int(math.sqrt(threshold/2))
                    new_height = int(new_width * height * 1.0 / width)
                else:
                    new_height = int(math.sqrt(threshold/2))
                    new_width = int(new_height * width * 1.0 / height)
                resized_im = im.resize((new_width, new_height))
                output_filename = filename.replace(source_dir, target_dir)
                resized_im.save(output_filename)
```

这里是压缩的参数设置：

```
source_dir = "input"
target_dir = "output"
threshold = 2000000
```

这里是拼接图片的代码：

```
#####################################################
# Notice !                                          #
# This script file should be placed in the same     #
# folder as the image.                              #
#####################################################

import sys, os, shutil, math
from PIL import Image

#####################################################
# parameter setting                                 #
#####################################################
bol_auto_place = True                     # auto place the image as a squared image， if 'True', ignore var 'row' and 'col' below
row            = 4                         # row number which means col number images per row
col            = 8                         # col number which means row number images per col
nw             = 400                       # sub image size, nw x nh
nh             = 400

path = os.getcwd();          # acquire current folder path

if os.path.exists('tmp'):    # ensure the 'tmp' folder is empty
   shutil.rmtree('tmp')
os.makedirs('tmp')

file_ls = os.listdir()       # list all files in this folder
file_ls.sort(key=len)

i = 0                        # a counter for images
for file in file_ls:
	name, extension = os.path.splitext(file);    # get file info[name, extension]
	if (extension == '.png' or extension == '.jpg' or extension == '.jpeg') and name != 'splicing_picture':    # select the image
		i += 1                               # image counter++
		print('%s...%s%s' % (i, name, extension))
		os.chdir(path)                       # ensure the image folder in every loop
		im = Image.open(file)                # open the image
		w, h = im.size                       # get image info
		#print('Original image size: %sx%s' % (w, h))
		if nw == nh:                         # if image should be 1:1 size
			if w >= h:
				box = ((w - h) // 2, 0, (w + h) // 2, h)
			else:
				box = (0, (h - w) // 2, w, (h + w) // 2)
			region = im.crop(box)            # crop the image to 1:1 and keep center region
		else:
			region = im                      # do nothing
		sname = '%s%s' % (str(i), '.png')    # rename 'x.png', x is a number from 1 to N
		os.chdir('tmp')                      # get into the folder 'tmp'
		region.save(sname, 'png')            # save the square image

os.chdir(path)        # ensure the path
os.chdir('tmp')

if bol_auto_place:    # auto place a big 1:1 square image 
	row = math.ceil(i ** 0.5)
	col = math.ceil(i ** 0.5)

dest_im = Image.new('RGBA', (col * nw, row * nh), (255, 255, 255))    # the image size of splicing image, background color is white

for x in range(1, col + 1):          # loop place the sub image
	for y in range(1,row + 1):
		try:
			src_im = Image.open("%s.png" % str( x + ( y - 1 ) * col))  # open files in order
			resize_im = src_im.resize((nw, nh), Image.ANTIALIAS)       # resize again
			dest_im.paste(resize_im, ((x-1) * nw, (y-1) * nh))         # paste to dest_im
		except IOError:
			pass

os.chdir(path)        # ensure the path
shutil.rmtree('tmp')  # delete the 'tmp'

dest_im.save('splicing_picture.png', 'png')
dest_im.show()        # finish


```