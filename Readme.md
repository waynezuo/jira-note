Для "лицензирования" Jira 7.5.0 необходимо внести изменения в два файла (приведены актуальные версии файлов на момент установки):
   
```
     atlassian-extras-3.2.jar
     atlassian-universal-plugin-manager-plugin-2.22.5.jar
``` 

Месторасположение файлов соответственно:
   
```
     /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/atlassian-extras-3.2.jar
     /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.5.jar
```

Скачиваем файлы на локальный компьютер, устанавливаем инструмент JD-GUI (http://jd.benow.ca/). Далее:

   - открываем atlassian-extras-3.2.jar с помощью декомпилятора JD-GUI
   - жмем File -> Save All Sources или Ctrl+Alt+S (сохранится архив atlassian-extras-3.2.jar.src.zip)
   - распаковываем полученный архив с помощью архиватора
   - в atlassian-extras-3.2.jar.src/com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java находим метод loadLicenseConfiguration (в моем случае выглядел так):
```
     /*     */   private Properties loadLicenseConfiguration(Reader text)
     /*     */   {
     /*     */     try
     /*     */     {
     /* 218 */       Properties props = new Properties();
     /* 219 */       new DefaultPropertiesPersister().load(props, text);
     /* 220 */       return props;
     /*     */     }
     /*     */     catch (IOException e)
     /*     */     {
     /* 224 */       throw new LicenseException("Could NOT load properties from reader", e);
     /*     */     }
     /*     */   }
```

   - и добавляем в данный метод информацию о лицензии:
```
     /*     */   private Properties loadLicenseConfiguration(Reader text)
     /*     */   {
     /*     */     try
     /*     */     {
     /* 218 */       Properties props = new Properties();
     /* 219 */       new DefaultPropertiesPersister().load(props, text);

                     props.setProperty("LicenseExpiryDate", "2031-01-01");
                     props.setProperty("MaintenanceExpiryDate", "2031-01-01");
                     props.setProperty("Evaluation", "false");
                     props.setProperty("NumberOfUsers", "-1");
                     props.setProperty("Organisation", "TerriconMembers");
                     props.setProperty("PurchaseDate", "2017-01-01");
                     props.setProperty("SEN", "SEN-L10306871");

     /* 220 */       return props;
     /*     */     }
     /*     */     catch (IOException e)
     /*     */     {
     /* 224 */       throw new LicenseException("Could NOT load properties from reader", e);
     /*     */     }
     /*     */   }
```
   - сохраняем файл
   - копируем commons-codec-1.9.jar в директорию с исходниками (atlassian-extras-3.2.jar.src)
   - переходим в каталог с исходниками (atlassian-extras-3.2.jar.src) и компилируем класс из java-файла, который мы правили:
```
     javac -cp commons-codec-1.9.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java 
```
   - могут быть ошибки (вызваны “кривостью” декомпиляции исходников), например:
```
     ./com/atlassian/extras/common/LicenseException.java:7: error: class, interface, or enum expected
     /*    */  * @deprecated
               ^
     ./com/atlassian/extras/common/LicenseException.java:8: error: class, interface, or enum expected
     /*    */  */
               ^
     ./com/atlassian/extras/common/org/springframework/util/StringUtils.java:101: error: variable strLen is already defined in method hasText(String)
     /*     */     int strLen;
                       ^
     ./com/atlassian/extras/common/org/springframework/util/StringUtils.java:486: error: variable strLen is already defined in method changeFirstCharacterCase(boolean,String)
     /*     */     int strLen;
                       ^
     Note: ./com/atlassian/extras/common/org/springframework/util/StringUtils.java uses unchecked or unsafe operations.
     Note: Recompile with -Xlint:unchecked for details.
```
   - устраняем ошибки и еще раз компилируем класс. После успешного выполнения в каталоге com/atlassian/extras/decoder/v2/ появится файл Version2LicenseDecoder.class
   - полученныей файл копируем с заменой (по такому же пути com/atlassian/extras/decoder/v2/) в архив atlassian-extras-3.2.jar (проще всего это сделать через mc - Midnight Commander)
   - в общем случае “новый” архив atlassian-extras-3.2.jar необходимо положить (с заменой) на сервере с Jira в каталог /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/ и перезапустить жиру, в нашем случае нужно пересобрать docker-образ согласно инструкциям в Dockerfile
   
   
Аналогичным образом патчим файл atlassian-universal-plugin-manager-plugin-2.22.5.jar:

   - открываем atlassian-universal-plugin-manager-plugin-2.22.5.jar с помощью декомпилятора JD-GUI
   - жмем File -> Save All Sources или Ctrl+Alt+S (сохранится архив atlassian-universal-plugin-manager-plugin-2.22.5.jar.src.zip)
   - распаковываем полученный архив с помощью архиватора
   - в atlassian-universal-plugin-manager-plugin-2.22.5.jar.src/com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java находим метод loadLicenseConfiguration (в моем случае выглядел так):
```
       private Properties loadLicenseConfiguration(Reader text)
       {
         try
         {
           Properties props = new Properties();
           new DefaultPropertiesPersister().load(props, text);
           return props;
         }
         catch (IOException e)
         {
           throw new LicenseException("Could NOT load properties from reader", e);
         }
       }
```       
       
   - и добавляем в данный метод информацию о лицензии:
```
       private Properties loadLicenseConfiguration(Reader text)
       {
         try
         {
           Properties props = new Properties();
           new DefaultPropertiesPersister().load(props, text);
           props.setProperty("LicenseExpiryDate", "2031-01-01");
           props.setProperty("MaintenanceExpiryDate", "2031-01-01");
           props.setProperty("Evaluation", "false");
           props.setProperty("NumberOfUsers", "-1");
           return props;
         }
         catch (IOException e)
         {
           throw new LicenseException("Could NOT load properties from reader", e);
         }
       }
```
   - сохраняем файл
   - копируем commons-codec-1.9.jar в директорию с исходниками (atlassian-universal-plugin-manager-plugin-2.22.5.jar.src)

   - переходим в каталог с исходниками (atlassian-universal-plugin-manager-plugin-2.22.5.jar.src) и компилируем класс из java-файла, который мы правили:
```
     javac -cp commons-codec-1.9.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```     
   - могут быть ошибки (вызваны “кривостью” декомпиляции исходников), устраняем их и еще раз компилируем класс. После успешного выполнения в каталоге com/atlassian/extras/decoder/v2/ появится файл Version2LicenseDecoder.class
   - полученныей файл копируем с заменой (по такому же пути com/atlassian/extras/decoder/v2/) в архив atlassian-universal-plugin-manager-plugin-2.22.5.jar (проще всего это сделать через mc - Midnight Commander)
   - в общем случае “новый” архив atlassian-universal-plugin-manager-plugin-2.22.5.jar необходимо положить (с заменой) на сервере с Jira в каталог /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/, удалить каталоги ${JIRA_HOME}/plugins/.bundled-plugins и ${JIRA_HOME}/plugins/.bundled-plugins/.osgi-plugins и перезапустить жиру, в нашем случае нужно пересобрать docker-образ согласно инструкциям в Dockerfile
