### If you are looking for ways to hide some specific rows and columns from specific users in qlik sense, the following could help you.

* **step1**:set user's access to _**data**_ on row and column level

```
section access;
LOAD * inline [
ACCESS, USERID,REDUCTION, OMIT    #reduction:define which row to hide, omit:define which column to hide
USER, AD_DOMAIN\ADMIN,,
USER, AD_DOMAIN\A,1, 
USER, AD_DOMAIN\B, 2,NUM
USER, AD_DOMAIN\C, 3, ALPHA
ADMIN, INTERNAL\SA_SCHEDULER,*,
];
section application;
```


note: here we only hide measurement, not dimensions(the report will have visual problems if we omit dimension, details pls refer to qlik help website)
 
 
* **step2**: afer step1, the colunm will still stay in the report for those specific users, but the measurement all equals = 0,which means the first step works but not very user friendly, and sometimes biz don't want users aware of there are autority difference among them, so here is step2 adjustment on report level.
 
![image](https://github.com/gege521/Qlik-Sense/blob/master/report.png)
 
We set _**show column if**_ measurement equals zero in the menu as the above

Combined 1&2, we can get a user friendly report which could hide some rows&columns to speccific users.

