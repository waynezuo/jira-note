To "license" Jira 7.5.0, you need to make changes to two files (the actual versions of the files are given at the time of installation):
   
```
     atlassian-extras-3.2.jar
     atlassian-universal-plugin-manager-plugin-2.22.5.jar
``` 

File locations respectively:
   
```
     /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/atlassian-extras-3.2.jar
     /opt/atlassian/jira/atlassian-jira/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.5.jar
```

Download the files to the local computer, install the JD-GUI tool (http://jd.benow.ca/). Further:

   - open atlassian-extras-3.2.jar with JD-GUI decompiler
   - click File -> Save All Sources or Ctrl+Alt+S (the atlassian-extras-3.2.jar.src.zip archive will be saved)
   - unpack the resulting archive using the archiver
   - in atlassian-extras-3.2.jar.src/com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java we find the loadLicenseConfiguration method (in my case it looked like this):
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

   - and add license information to this method:
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
   - save the file
   - copy commons-codec-1.9.jar to source directory (atlassian-extras-3.2.jar.src)
   - go to the source directory (atlassian-extras-3.2.jar.src) and compile the class from the java file that we edited:
```
     javac -cp commons-codec-1.9.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java 
```
   - there may be errors (caused by the “crooked” decompilation of source codes), for example:
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
   - fix the errors and compile the class again. After successful execution, Version2LicenseDecoder.class file will appear in the com/atlassian/extras/decoder/v2/ directory
   - copy the resulting file with replacement (along the same path com/atlassian/extras/decoder/v2/) to the atlassian-extras-3.2.jar archive (the easiest way to do this is through mc - Midnight Commander)
   - in general, the “new” atlassian-extras-3.2.jar archive must be placed (with replacement) on the server with Jira in the /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/ directory and restart the jira, in our in case you need to rebuild the docker image according to the instructions in the Dockerfile
   
   
Similarly, we patch the atlassian-universal-plugin-manager-plugin-2.22.5.jar file:

   - open atlassian-universal-plugin-manager-plugin-2.22.5.jar with JD-GUI decompiler
   - click File -> Save All Sources or Ctrl+Alt+S (the atlassian-universal-plugin-manager-plugin-2.22.5.jar.src.zip archive will be saved)
   - unpack the resulting archive using the archiver
   - in atlassian-universal-plugin-manager-plugin-2.22.5.jar.src/com
	/atlassian/extras/decoder/v2/Version2LicenseDecoder.java we find the loadLicenseConfiguration method (in my case it looked like this):
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
       
   - and add license information to this method:
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
   - save the file
   - copy commons-codec-1.9.jar to source directory (atlassian-universal-plugin-manager-plugin-2.22.5.jar.src)

   - go to the source directory (atlassian-universal-plugin-manager-plugin-2.22.5.jar.src) and compile the class from the java file that we edited:
```
     javac -cp commons-codec-1.9.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```     
   - there may be errors (caused by the “crooked” decompilation of the sources), we eliminate them and compile the class again. After successful execution, the Version2LicenseDecoder.class file will appear in the com/atlassian/extras/decoder/v2/ directory
   - copy the resulting file with replacement (along the same path com/atlassian/extras/decoder/v2/) to the atlassian-universal-plugin-manager-plugin-2.22.5.jar archive (the easiest way to do this is through mc - Midnight Commander)
   - in general, the “new” atlassian-universal-plugin-manager-plugin-2.22.5.jar archive must be placed (with replacement) on the Jira server in the /opt/atlassian/jira/atlassian-jira/WEB-INF/ directory atlassian-bundled-plugins/, delete ${JIRA_HOME}/plugins/.bundled-plugins and ${JIRA_HOME}/plugins/.bundled-plugins/.osgi-plugins directories and restart jira, in our case we need to rebuild the docker image according to instructions in Dockerfile
