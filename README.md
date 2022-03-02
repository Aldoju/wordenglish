# wordenglish

from cgitb import text
from sre_constants import IN
from tkinter import *  #para no incluir el sufijo t cunado importo de la libreria tkinter
from tkinter import ttk
from tkinter import scrolledtext  
from tkinter.ttk import LabelFrame, Notebook  
import tkinter as t

import sqlite3


  

class Ventana:
    BD="WordBD.db"
    
    
    
    def __init__(self,window):
        self.wind=window
        self.wind.title("Vocabulary")
        self.wind.geometry("750x400")
        
        
        frame=LabelFrame(self.wind)
        frame.grid(row=2,column=0,columnspan=3,padx=150,pady=50)

        # word
        t.Label(frame,text="Word").grid(row=3,column=1)
        self.word=t.Entry(frame)
        self.word.focus()
        self.word.grid(row=3,column=2)
        #English num
        t.Label(frame,text="English Num").grid(row=4,column=1)
        self.num=t.Entry(frame)
        self.num.grid(row=4,column=2)
        
        #Unit
        Label(frame,text="Unit").grid(row=5,column=1)
        self.unit=Entry(frame)
        self.unit.grid(row=5,column=2)
        
        
        #Mean
        t.Label(frame,text="Mean").grid(row=6,column=1)
        self.mean=t.Entry(frame)
        self.mean.grid(row=6,column=2)
        
        #buttons
        t.Button(frame,text="Add word",command=lambda:self.addword(var)).grid(row=3,column=3,sticky=W + E,padx=10)
        t.Button(frame,text="Search word").grid(row=4,column=3,sticky=W + E,padx=10)
        t.Button(frame,text="Delete word",command=lambda:self.deleteword(var)).grid(row=5,column=3,sticky=W + E,padx=10)
        
        #command=lambda ---me permite agregar parametros a las funcion que se ejecuta al presionar el boton
        
        #mensaje salida
        self.message=Label(frame,text="",fg="red")
        self.message.grid(row=7,column=1,sticky=W + E)
    
        #ventana para elegir tabla de English 
        Label(self.wind,text="ENGLISH").grid(row=8,column=1)
        var=IntVar(self.wind)
        var.set(1)

        opciones=[ 1, 2 , 3 , 4 , 5 , 6 ] 
        mul=OptionMenu(self.wind,var,*opciones,command=self.getproduct)    # * en opciones es necesario para referenciar varias opciones del 1 al 6 
        mul.grid(row=8,column=2)


        
        #tables
        self.tree=ttk.Treeview(self.wind,height=10,columns=[f"#{n}" for n in range(1, 2)])
        self.tree.grid(row=10,column=1,padx=10,pady=10)
        self.tree.heading("#0",text="Word",anchor=CENTER)
        self.tree.heading("#1",text="Unit",anchor=CENTER)
        textar=scrolledtext.ScrolledText(self.wind,height=10,width=40)
        textar.grid(row=10,column=2,padx=10,pady=10)
        self.getproduct(var)
    
    def runquery(self,query,parameters=()):
            with sqlite3.connect(self.BD) as con:
                cursor=con.cursor()
                result=cursor.execute(query,parameters)
                con.commit()
            return result
        
    def getproduct(self,var):
        record=self.tree.get_children()
        for element in record:
            self.tree.delete(element)
        if var==1:
            query="SELECT* from ENGLISH Where englishNum=1"
        elif var==2:
            query="SELECT* from ENGLISH Where englishNum=2"
        elif var==3:
            query="SELECT* from ENGLISH Where englishNum=3"
        elif var==4:
            query="SELECT* from ENGLISH Where englishNum=4"
        elif var==5:
            query="SELECT* from ENGLISH Where englishNum=5"
        elif var==6:
            query="SELECT* from ENGLISH Where englishNum=6"
        else:
            query="SELECT* from ENGLISH Where englishNum=1"  # para evitar error de query referenced before asignment
            
        db_rows=self.runquery(query)
        for row in db_rows:
            self.tree.insert("",0,text=row[1],values=row[2])
            
            
    def validation(self):
        return len(self.word.get())!=0 and len(self.num.get())!=0 and len(self.unit.get())!=0 and len(self.mean.get())!=0 
    

    def addword(self,var):
        if self.validation:
            query="INSERT INTO ENGLISH VALUES(?,?,?,?)"
            parameters=(self.num.get(),self.word.get(),self.unit.get(),self.mean.get())
            self.runquery(query,parameters)
            self.message["text"]="Word {} added successfully".format(self.word.get())
            self.num.delete(0,END)
            self.word.delete(0,END)
            self.unit.delete(0,END)
            self.mean.delete(0,END)
            self.getproduct(var)
        else:
            self.message["text"]="fill with all the neccesary information please"
            
    def deleteword(self,var):
        self.message['text']=''
        try:
            self.tree.item(self.tree.selection())['text'][0]           #se obtine la primera letra de word,se verifica que se seleccione la palabra
        except IndexError as e:
            self.message["text"]='Select some word to delete'
            return
        self.message["text"]=''
        wordel=self.tree.item(self.tree.selection())['text']
        query="DELETE FROM ENGLISH WHERE word=?"
        self.runquery(query,(wordel,))
        self.message['text']="word {} deleted successfully ".format(wordel)
        self.getproduct(var)            
    
    
    
if __name__=="__main__":
    window=t.Tk()
    appliction=Ventana(window)
    window.mainloop()
    
    



