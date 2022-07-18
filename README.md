# getting-and-cleaning-data-project
proyecto final del curso getting and cleaning data
#este es el codigo utilizado para el proyecto
#primero lo hice en la consola de Rstudio
library(dplyr)

xtrain<-read.table('./UCI HAR Dataset/train/X_train.txt', header=FALSE)
ytrain<-read.table('./UCI HAR Dataset/train/y_train.txt', header=FALSE)
#luego de obtener los archivos xtrain y ytrain habiendolos ubicados en el directorio de busqueda
#se realiza lo mismo con el archivo features
features<-read.table('./UCI HAR Dataset/features.txt', header=FALSE)
#adicional a eso con los otros archivos que estaban en el zip
xtest<-read.table('./UCI HAR Dataset/test/X_test.txt', header=FALSE)
ytest<-read.table('./UCI HAR Dataset/test/y_test.txt', header=FALSE)
activity<-read.table('./UCI HAR Dataset/activity_labels.txt', header=FALSE)
#posteriormente se deben extraer los archivos subject
subtrain<-read.table('./UCI HAR Dataset/train/subject_train.txt', header=FALSE)
subtrain<-subtrain%>%
rename(subjectID=V1)
subtest<-read.table('./UCI HAR Dataset/test/subject_test.txt', header=FALSE)
subtest<-subtest%>%
rename(subjectID=V1)

#bueno ahora si, se añaden columnas
features<-features[,2]
featrasp<-t(features)
#se cambian nombres de columnas 
colnames(xtrain)<-featrasp
colnames(xtest)<-featrasp
#se renombra la actividad
colnames(activity)<-c('id','actions')
#la tarea es combinar los objetos de entrenamiento train, asi que por columnas se realiza
combineX<-rbind(xtrain, xtest)   
combineY<-rbind(ytrain, ytest)
combineSubj<-rbind(subtrain,subtest)
#una vez combinados tendremos entonces un nuevo data frame puedo entonces realizar un merge
YXdf<-cbind(combineY,combineX, combineSubj)

df<-merge(YXdf, activity,by.x = 'V1',by.y = 'id')

#un vez asi puedo obtener media y desviacion estandar
colNames<-colnames(df)
df2<-df%>%
   select(actions, subjectID, grep("\\bmean\\b|\\bstd\\b",colNames))
#realizzo un cambio de naturaleza de variables para hacerlas factor (de lo contrario no puedo sacar media)
df2$actions<-as.factor(df2$actions)

colnames(df2)<-gsub("^t", "time", colnames(df2))
colnames(df2)<-gsub("^f", "frequency", colnames(df2))
colnames(df2)<-gsub("Acc", "Accelerometer", colnames(df2))
colnames(df2)<-gsub("Gyro", "Gyroscope", colnames(df2))
colnames(df2)<-gsub("Mag", "Magnitude", colnames(df2))
colnames(df2)<-gsub("BodyBody", "Body", colnames(df2))
#creo un nuevo set con todos los datos de la actividad
df2.2<-aggregate(. ~subjectID + actions, df2, mean)

  
write.table(df2.2, file = "tidydata.txt",row.name=FALSE)
