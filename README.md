

import glob
import math
from pathlib import Path
import pandas as pd
import re
import matplotlib.pyplot as plt
import matplotlib.image as mpimg
import matplotlib.patches as patches
import numpy as np
import os

temp = []


def transformCordinates(coordinates,wmax,hmax):
    coordarr=coordinates.split()
    coordarr = [float(x) for x in coordarr]
    a=coordarr[0]
    b=coordarr[1]
    theta=coordarr[2]
    xcenter=coordarr[3]
    ycenter=coordarr[4]
    xo=(a**2*math.cos(theta)**2+b**2*math.sin(theta)**2)**0.5
    yo=(b**2*math.cos(theta)**2+a**2*math.sin(theta)**2)**0.5
    yo2=2*yo
    xo2=2*xo
    x=xcenter-xo
    y=ycenter-yo
    width=xo2
    heigth=yo2
    rec0 = [x,y,width,heigth]
    return (rec0)

def pdToXml(name, coordinates, size, img_folder):
    xml = ['<annotation>']
    xml.append("    <folder>{}</folder>".format(img_folder))
    xml.append("    <filename>{}</filename>".format(name))
    xml.append("    <source>")
    xml.append("        <database>Unknown</database>")
    xml.append("    </source>")
    xml.append("    <size>")
    xml.append("        <width>{}</width>".format(size["width"]))
    xml.append("        <height>{}</height>".format(size["height"]))
    xml.append("        <depth>3</depth>".format())
    xml.append("    </size>")
    xml.append("    <segmented>0</segmented>")

    for field in coordinates:
        xmin, ymin = max(0,field[0]), max(0,field[1])
        xmax = min(size["width"], field[0]+field[2])
        ymax = min(size["height"], field[1]+field[3])

        xml.append("    <object>")
        xml.append("        <name>Face</name>")
        xml.append("        <pose>Unspecified</pose>")
        xml.append("        <truncated>0</truncated>")
        xml.append("        <difficult>0</difficult>")
        xml.append("        <bndbox>")
        xml.append("            <xmin>{}</xmin>".format(int(xmin)))
        xml.append("            <ymin>{}</ymin>".format(int(ymin)))
        xml.append("            <xmax>{}</xmax>".format(int(xmax)))
        xml.append("            <ymax>{}</ymax>".format(int(ymax)))
        xml.append("        </bndbox>")
        xml.append("    </object>")
    xml.append('</annotation>')
    return '\n'.join(xml)

def generateArray(file):
    with open(file, "r") as f:
        arr = f.read().splitlines()
    
    arr_len = len(arr)
    i = 0    
    # regex
    rg = re.compile("(\d)*_(\d)*_(\d)*_big")
    output = []
    while i != arr_len:
        val = arr[i] # nombre de la imagen
        mtch = rg.match(val)
        if mtch:
            try:
                di = dict() #diccionario
                val = "{}.jpg".format(val)
                di["name"] = val
                #  matplotlib
                img = mpimg.imread(os.path.join("dataset", val))
                fig,ax = plt.subplots(1)
                ax.imshow(img)               
                (h, w, _) = img.shape
                #rects=[]
                
                
                #for e in val
                #rects.append(transformCordinates(e, width,heigth))

                #labels.append({'name':val, "img":img,'size':{"width":width,"height":height},"faces":faces,"rects":rects})

                jumps = int(arr[i+1])
                temp = []
                for j in range(0, jumps):
                    coords = arr[i + j +2]
                    temp.append(coords)
                    rec = transformCordinates(coords,w,h)
                    rect = patches.Rectangle(
                        (rec[0],rec[1]),rec[2],rec[3],
                        linewidth=1,
                        edgecolor='r',
                        facecolor='none')
                    ax.add_patch(rect)
                plt.show()
                di["annotations"] = temp
                i+=jumps+1
            except Exception as e: 
                print(e)
                print("{} not found...".format(val))
                i+=1
                #return output
        else:
            i+=1



def returnEllipseListFiles(path):
    return [str(f) for f in Path(path).glob('**/*-ellipseList.txt')]

folder = glob.glob("dataset/*.jpg")
folder = pd.Series(folder)
files = returnEllipseListFiles("labels")

dictionary=[generateArray(f) for f in files]
