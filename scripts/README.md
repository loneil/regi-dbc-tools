## Scripts

### Metrics

Counts of issued DBCs etc

```sql
-- Count issued creds - Distinct users
select count(*) from dc_credentials dc 
inner join dc_business_users dbu on dbu.id = dc.business_user_id
where dc.is_issued = true 
and dc.is_revoked = false;

-- Count issued creds - Distinct users
select distinct dbu.user_id from dc_credentials dc 
inner join dc_business_users dbu on dbu.id = dc.business_user_id
where dc.is_issued = true 
and dc.is_revoked = false;

-- Count issued creds - Distinct businesses
select distinct dbu.business_id from dc_credentials dc 
inner join dc_business_users dbu on dbu.id = dc.business_user_id
where dc.is_issued = true 
and dc.is_revoked = false;

-- Count up issuances by day
select date_of_issue::date as day,
count(*) as total
from dc_credentials dc 
inner join dc_business_users dbu on dbu.id = dc.business_user_id
group by day
order by day asc;

-- Count up issueances by day (only include still active)
select date_of_issue::date as day,
count(*) as total
from dc_credentials dc 
inner join dc_business_users dbu on dbu.id = dc.business_user_id
where dc.is_issued = true 
and dc.is_revoked = false
group by day
order by day asc;


-- Select revoked
select * from dc_credentials dc 
inner join dc_business_users dbu on dbu.id = dc.business_user_id
where dc.is_revoked = true;
```

### Get unique connection IDs for issued credential holders

Query a list of
- Connection IDs
- For issued credentials (exclude revoked)
- Mapping the business user for the credential and excluding any duplicated related USER Ids

So a user will not get duplicated messages if they have multiple credentials

```sql
SELECT DISTINCT ON (u.id) dcn.connection_id, u.id
FROM dc_credentials dc
INNER JOIN dc_business_users dbu ON dbu.id = dc.business_user_id
INNER JOIN users u ON dbu.user_id = u.id
INNER JOIN dc_connections dcn ON dcn.business_user_id = dbu.id
WHERE dc.is_issued = true
AND dc.is_revoked = false
ORDER BY u.id, dcn.connection_id;
```