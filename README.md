# first_file

# Учебное задание по обработке данных, приведении их в формат PDF и отправки их сообщением  cо вставкой HTML-кода.
## подгрузили необходимые библиотеки для дальнейшей работы
```
%matplotlib inline   
import binascii   
import pandas as pd  
import matplotlib.pyplot as plt  
import pdfkit   
import smtplib   
import mimetypes   
from io import BytesIO  
from email import encoders   
from email.mime.text import MIMEText   
from email.mime.multipart import MIMEMultipart      
from email.mime.application import MIMEApplication   
from email.mime.base import MIMEBase   
```
## обработали данные с помощью библиотеки Pandas
```
data = pd.read_csv('https://video.ittensive.com/python-advanced/data-9722-2019-10-14.utf.csv', delimiter = ';')  
data = data.dropna(axis=1)   
data['District'] = data['District'].str.replace('район','').astype('category')   
data['AdmArea'] = data['AdmArea'].apply(lambda x: x.split(' ')[0]).astype('category')   
data = data.set_index('YEAR').loc['2018-2019'].reset_index()   
data_best = data.sort_values('PASSES_OVER_220', ascending = False).head(1)   

all_student = data.set_index('AdmArea')    
all_student = all_student['PASSES_OVER_220'].groupby('AdmArea').sum().sort_values()  
total =all_student.sum()  
pd.options.display.max_colwidth = 1000   
```
## построили круговую диаграмму с отображением наших данных
```
fig = plt.figure(figsize=(12,6))  
area = fig.add_subplot(1, 1, 1)  
explode = [0]*len(all_student)   
explode[0] = 0.4  
explode[1] = 0.4  
all_student.plot.pie(ax=area,labels=['']*len(all_student),    
                             label = 'Отличники по округам',    
                             cmap ='tab20',   
                             autopct=lambda x:int(round(total * x/100)),  
                             pctdistance=0.9,  
                             explode=explode)  
area.legend(all_student.index, bbox_to_anchor=(1.5,1,0.1,0))  
plt.savefig(home_work.png)  
```
##  преобразовали файл в формат base64
```
with open('home_work.png', 'rb') as file:   
    img = 'data:image/png;base64,' + binascii.b2a_base64(file.read(), newline=False).decode('UTF-8')  
```
## пишем HTML-код 
```
html = '''<html>  
<head>  
    <title>Количество отличников по округам Москвы</title>  
    <meta charset='utf-8'/>  
</head>  
<body>  
    <h1 style='background:#666;padding:10px;color:#fff'>  
        Количество отличников по округам Москвы в 2018-2019 годах  
    </h1>  
    <p>Всего отличников: ''' + str(total) + ''' </p>  
    <img src="'''+ img + '''" alt='Отличники по округам'/>  
    <p>Лучшая школа: ''' + str(data_best['EDU_NAME'].values[0]) +'''</p>  
</body>  
</html>'''  
```

##  формируем PDF-документ
```
config = pdfkit.configuration(wkhtmltopdf='C:/Program Files/wkhtmltopdf/bin/wkhtmltopdf.exe')  
options ={  
    'page-size':'A4',   
    'header-right' :'[page]',   
    'enable-local-file-access': True,  
    "encoding": "utf-8"  
}  
pdfkit.from_string(html, 'img.pdf',   
                 configuration=config, options=options)   
```
## формируем сообщение и отправляем его адресату
```
sender= '#######'  
password = '######'  
    
server = smtplib.SMTP_SSL('smtp.yandex.com', 465)   
server.login(sender,password)                 
msg = MIMEMultipart()   
msg['From'] = '#######'  
msg['To'] = '#######'  
msg['Subject'] = 'Продвинутый Python, Домашнее задание.Выполнено'   

msg.attach(MIMEText(html,'html'))  
attachment = MIMEBase('application', 'pdf')  

attachment.set_payload(open('img.pdf', 'rb').read())   
attachment.add_header('content-disposition','attachment', filename="img.pdf")  

encoders.encode_base64(attachment)  
msg.attach(attachment)  
server.sendmail(sender,'support@ittensive.com', msg.as_string())   
server.quit()   
```
