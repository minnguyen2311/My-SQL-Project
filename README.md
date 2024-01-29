# Lucky Hat Game

Lucky Hat Game is a game designed by admins where users can join and win prizes based on their luck.

## Description
Below is the rules of the game:

* Every week users can play the game	
* Admin will decide the total times all users can play or total points of the game for each week
* Game will be stopped when total tries are met or total points have reached - Users reached the maximum tries can no longer play for that week
* Admin can decide the points and the times they appear on the scale, and the Quota users can get from the game (how many times the points appear- appear times of the points on the scale)
* Users can join the game 2 tries everyday
* Each time user gets a specific point for their profile
* Points will be calculated each week
* The game has Quota for the prize each week
* There is a SUPER PRIZE for winner of the month: based on the total point of month 

## SQL Queries

1. Show games and their values that user will see?

```bash
select
  g.name,
  p.value
from game g
  join `point` p on g.id = p.game_id;
```
2. What POINT NUMBER  in what games  is meet the quota (THE TIMES PLAYER GOT THE POINT OF THE GAME = QUOTA )?

```bash
select
	p.value, g.name, remaining_quota
from point p
	join game g on p.game_id = g.id
where remaining_quota = 0;
```

3. Show current game (GAME IS RUNNING NOT FINISHED YET ) and its details ?

```bash
select
	g.name, p.value
from game g
	join point p on g.id = p.game_id
where status = 'PROCESSING';
```
4. Who play MORE THAN 2 different games ?

```bash
select
	u.id, u.name, count(distinct game_id) as game_played
from user u
	join user_play_game upg on u.id = upg.user_id
group by u.id
having game_played > 2;
```

5. Who never win any prize?

```bash

select * 
from user 
where id not in (select distinct u.id
				from winner w
				join user u on w.user_id = u.id 
				UNION 
				select distinct u.id
				from monthly_winner w
				join user u on w.user_id = u.id) ;
```

6. What game have the BIGGEST AND SMALLEST number of prizes?

```bash
select
	g.id, count(*) as total_prize
from prize p
	join game g on p.game_id = g.id
group by g.id
having total_prize = (select max(total_prize) 
				from (select g.id, count(*) as total_prize
				from prize p
					join game g on p.game_id = g.id
				group by g.id) as temp_table)
                
UNION 

select
	g.id, count(*) as total_prize
from prize p
	join game g on p.game_id = g.id
group by g.id
having total_prize = (select min(total_prize) 
				from (select g.id, count(*) as total_prize
				from prize p
					join game g on p.game_id = g.id
				group by g.id) as temp_table);
```

7. What games have total play times more than average total play times (of all games)?
   
```bash
select
	g.id, g.name, count(*) as total_played_times
from user_play_game upg
	join game g on upg.game_id = g.id
group by g.id
having total_played_times > (select count(*)/count(distinct game_id)
								from user_play_game);
```
8. Show top 3 total points of all users all the times?

```bash
select
	u.id, u.name, sum(upg.point) as total_point
from user_play_game upg
	join user u on u.id = upg.user_id
group by u.id 
order by total_point desc
limit 3;
```

9. Show top 2 game have the least number of customer take part in?

```bash
select
	g.id, g.name, count(distinct user_id) as total_user
from user_play_game upg 
	join game g on g.id = upg.game_id
group by upg.game_id
order by total_user asc
limit 2;
```

10. Show ALL PRIZES  and their winner.

```bash
select
	g.name, p.name, u.name
from prize p 
	join game g on g.id = p.game_id
    join winner w on w.prize_id = p.id
    join user u on u.id = w.user_id;
```

11. What games have biggest and smallest AVERAGE POINT of all play times?

```bash
select
	game_id, avg(point) as average_point
from user_play_game
group by game_id
order by average_point asc 
limit 1

UNION 

select
	game_id, avg(point) as average_point
from user_play_game
group by game_id
order by average_point desc 
limit 1;
```

12. Show total playing times and point of each month ?
    
```bash
select
	YEAR(play_time) as year, MONTH(play_time) as month, sum(point) as total_point, count(*) as total_played_times
from user_play_game
group by YEAR(play_time), MONTH(play_time);
```

13. How many players of nearest month ?
    
```bash    
select
	count(distinct user_id)
from user_play_game
where TIMESTAMPDIFF(MONTH, play_time, now()) <1; 
```

14. Who are the winners and the prizes of the latest games ?
    
```bash    
select
	u.name, p.name, p.game_id
from winner w 
	join prize p on w.prize_id = p.id
    join user u on w.user_id = u.id
where p.game_id = (select id
				from game 
				where status = 'END'
				order by end_time desc
				limit 1);
```
       
15. SHOW THE APPEARANCE percentage of all POINTS EACH MONTH from HIGH TO LOW?

```bash     
select
	YEAR(start_time) as year , MONTH(start_time) as month, p.value as point, sum(appear_time) as appear_time, temp_table.total , sum(appear_time)*100/temp_table.total as percentage
from point p
	join game g  on p.game_id = g.id
    join (select YEAR(start_time) as year, MONTH(start_time) as month, sum(appear_time) as total
		from point p
		join game g  on p.game_id = g.id
		group by  YEAR(start_time), MONTH(start_time)) temp_table ON temp_table.year = YEAR(start_time) AND temp_table.month = MONTH(start_time)
group by p.value, YEAR(start_time), MONTH(start_time);
```

17. FIND TOP 2 months in the year attracts most people play game ?
    
```bash
select
	YEAR(play_time) as year, MONTH(play_time) as month, count(distinct user_id) as total_players
from user_play_game 
group by YEAR(play_time), MONTH(play_time)
order by total_players desc
limit 2;
```
