# SQL Murder Mystery

Solution and procedures for the SQL-based game [SQL Murder Mystery by by Joon Park and Cathy He](https://mystery.knightlab.com)

## Clues and Findings

***Eduardo Solano***

```sql
SELECT * FROM crime_scene_report 
where type='murder' and city='SQL City' and date='20180115'
```
 date | type | description | city 
---|---|---|---
 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City 

### Witnesses
1. Morty Schapiro
    ``` sql
    select * from person 
    where address_street_name='Northwestern Dr'
    ORDER BY address_number desc limit 1
    ```
    id | name | license_id | address_number | address_street_name | ssn 
    ---|---|---|---|---|---
    14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949 
 
 1. Annabel Miller
    ```sql
    select * from person 
    where address_street_name='Franklin Ave' and name like 'Annabel%' 
    ORDER BY address_number desc limit 1
    ```
    id | name | license_id | address_number | address_street_name | ssn 
    ---|---|---|---|---|---
    16371 | Annabel Miller | 490173 | 103 | Franklin Ave | 318771143 

### Interview transcripts

```sql
select p.name, i.transcript 
from person p join interview i on p.id=i.person_id 
where p.id in (16371, 14887)
```

 name | transcript 
---|---
 Morty Schapiro | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". 
 Annabel Miller | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th. 

 Kiler findings:

 1. Member in the same gym as Anabelle Miller: "Get Fit Now Gym"
 2. Went to the gym in January 9th 2018
 3. Bag
    1. Membership started with "48Z"
    2. Gold member
 4. Ran away in car with plate including "H42W"

Look at members matching the id and drivers matching the plates

```sql
select * from get_fit_now_member 
where id like "48Z%" and membership_status='gold'
```

 id | person_id | name | membership_start_date | membership_status 
---|---|---|---|---
 48Z7A | 28819 | Joe Germuska | 20160305 | gold 
 48Z55 | 67318 | Jeremy Bowers | 20160101 | gold 

```sql
select * 
from drivers_license join person on drivers_license.id=person.license_id
where plate_number like "%H42W%"
```

 id | age | height | eye_color | hair_color | gender | plate_number | car_make | car_model | id | name | license_id | address_number | address_street_name | ssn 
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
 664760 | 21 | 71 | black | black | male | 4H42WR | Nissan | Altima | 51739 | Tushar Chandra | 664760 | 312 | Phi St | 137882671 
 423327 | 30 | 70 | brown | brown | male | 0H42W2 | Chevrolet | Spark LS | 67318 | Jeremy Bowers | 423327 | 530 | Washington Pl, Apt 3A | 871539279 
 183779 | 21 | 65 | blue | blonde | female | H42W0X | Toyota | Prius | 78193 | Maxine Whitely | 183779 | 110 | Fisk Rd | 137882671 


### Suspect 1

Jeremy Bowers is in both queries, following the lead:

 ```sql
 select * 
from get_fit_now_check_in c join get_fit_now_member m on c.membership_id=m.id
where c.check_in_date in (20180109, 20180115) and m.person_id=67318
```

 membership_id | check_in_date | check_in_time | check_out_time | id | person_id | name | membership_start_date | membership_status 
---|---|---|---|---|---|---|---|---
 48Z55 | 20180109 | 1530 | 1700 | 48Z55 | 67318 | Jeremy Bowers | 20160101 | gold 

 Sadly, according to his interview he was hired, so another killer

 Transcript: *I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.*

 **SQL Murder Mystery - Check Solution**
 > Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.

### Suspect 2

Findings:
1. Woman
2. High Income
3. Height 65''-67''
4. Red Hair
5. Drives Tesla Model S
6. Attended SQL Symphony 3 times in December 2017

Let's see who matches this characteristics:

```sql
select p.id, p.name, d.age, d.height, d.hair_color, d.gender, d.car_model, i.annual_income, f.event_name, f.date
from person p join drivers_license d on p.license_id=d.id
	join income i on p.ssn=i.ssn
	join facebook_event_checkin f on p.id=f.person_id
where d.height between 60 and 70
	and d.car_model = 'Model S'
	and d.hair_color = 'red'
	and f.date between 20171201 and 20171231
```

 id | name | age | height | hair_color | gender | car_model | annual_income | event_name | date 
---|---|---|---|---|---|---|---|---|---
 99716 | Miranda Priestly | 68 | 66 | red | female | Model S | 310000 | SQL Symphony Concert | 20171206 
 99716 | Miranda Priestly | 68 | 66 | red | female | Model S | 310000 | SQL Symphony Concert | 20171212 
 99716 | Miranda Priestly | 68 | 66 | red | female | Model S | 310000 | SQL Symphony Concert | 20171229 

 Miranda Priestly matches all the findings; but she was never interviewed

 ### Conslusions

 The murderer was **Jeremy Bowers**, who confessed in his interview. But he was hired by a woman by the name of **Miranda Priestly**

 **SQL Murder Mystery - Check Solution**
 > Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!
