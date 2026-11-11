| רכיב | איפה הוא רץ (הנהלה / כל node) | מה הוא עושה, במשפט אחד | אנלוגיה |
|---|---|---|---|
| API server |הנהלה | כל בקשה מ- CLUSTER עוברת דרכו|כמין פקיד קבלה ראשי |
| etcd |הנהלה |שומר נתונים |כל מידע בית החולים נשמר שם |
| scheduler | הנהלה| מחליט לכל בקשה באיזה שרת לרוץ|משבץ מיטות במחלקהס|
| controller manager |הנהלה |בודק אם הכל במצב פועל | פיקוח במחלקות שבכל במצב תקין|
| kubelet | כל NODE| מפעיל את הקונטיינר|אחראי המחלקה|
| container runtime | כל NODE|  התוכנה שבפועל | הצוות שבפועל|
| kube-proxy |כל NODE |מנהל את הכללים ברשת |שילוט |
| coredns | לא בהרכח על כל NODE|מתרגם סרברס ושמות IP | מקשרת בין מחלקות|	


1. 
- הפקודה kubectl שולחת את הבקשה ל-API server, שבודק אותה ומעדכן ב-etcd שה-Deployment צריך עכשיו 4 עותקים.
- ה-controller manager רואה שיש 2 pods אבל רצויים 4, ויוצר דרך ה-API server שני אובייקטים חדשים של pod. הם נשמרים ב-etcd, עדיין בלי node.
- ה-scheduler רואה pods בלי node, בוחר לכל אחד node מתאים ורושם את ההחלטה דרך ה-API server.

2. 
- כשהקונטיינר קורס: ה-kubelet מזהה שהקונטיינר מת ומפעיל אותו מחדש, לפי ה-restartPolicy. ברירת המחדל היא Always. זה אותו pod, עם אותו שם ואותה כתובת, ורק מונה ה-RESTARTS עולה.
- כשמוחקים את ה-pod כולו: ה-kubelet כבר לא יכול לעזור, כי אין pod להפעיל מחדש. אבל אם ה-backend מנוהל על ידי Deployment, ה-controller manager רואה שחסר pod ויוצר חדש.

3. 
- בענן: ה-kubelet ב-node 2 מפסיק לדווח שהוא חי. אחרי בערך 40 שניות ה-controller manager מסמן את ה-node כ-NotReady, ואחרי עוד כמה דקות (ברירת המחדל היא 5) ה-pods שעליו מסומנים למחיקה. ה-controllers של ה-Deployments רואים שחסרים pods ויוצרים חדשים, וה-scheduler משבץ אותם ב-node 1 וב-node 3. pod שנוצר בלי Deployment פשוט אובד.

- ואחרי עוד כמה דקות  ה-pods שעליו מסומנים למחיקה. ה-controllers של ה-Deployments רואים שחסרים pods ויוצרים חדשים, וה-scheduler משבץ אותם ב-node 1 וב-node 3. pod שנוצר בלי Deployment פשוט אובד.

- ב-Docker Desktop: יש רק node אחד, והוא גם ה-control plane. אם המחשב נכבה, כל ה-cluster נופל: אין לאן להעביר pods ואין מי שינהל את ההעברה. כשהמחשב עולה, ה-cluster עולה מחדש מהמצב שנשמר ב-etcd, וה-pods חוזרים.

4. 
- הקונטרולר רואה שחסר פוד אז הולך ומיצר חדש, והSCEDULER משבץ אותו והKUBELET מפעיל אותו.


5. 
- הפקודה kubectl run יוצרת pod בודד, בלי אף controller שאחראי עליו. אף אחד לא רשם ב-etcd ש"צריך להיות pod כזה", אז כשהוא נמחק, אין מי שישים לב שהוא חסר.
- ה-pod  של web שייך ל-Deployment, דרך ReplicaSet. ה-controller רואה שיש פחות pods ממה שרצוי ויוצר חדש.