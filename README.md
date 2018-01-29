## Welcome to DFA

You can use the [editor on GitHub](https://github.com/juice457/DFA/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### ERD

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Query Examples
```SQL


########################################################################################################################################################
##1. Dependecies
########################################################################################################################################################
##[1] Most used ports
SELECT port, count(port)
FROM df_expose 
WHERE current = true
GROUP BY port
ORDER BY count DESC

##[2] How many times do dependencies change
SELECT *
FROM diff_type NATURAL JOIN diff NATURAL JOIN snap_diff NATURAL JOIN snap_id
WHERE change_type LIKE '%Updat%' AND instruction = 'RUN'

##[3.1] Which images are preferred
SELECT imagename, count(imagename)
FROM df_from 
WHERE current = true
GROUP BY imagename
ORDER BY count DESC

##[3.2] Which images version are preferred (NUMERIC) 
SELECT imageversionnumber, count(imageversionnumber)
FROM df_from 
WHERE current = true
GROUP BY imageversionnumber
ORDER BY count DESC

##[3.3] Which images version are preferred  (NOMINAL)
SELECT imageversionstring, count(imageversionstring)
FROM df_from 
WHERE current = true
GROUP BY imageversionstring
ORDER BY count DESC


##[4] How many images use the :latest tag?
SELECT imageversionnumber, count(imageversionnumber)
FROM df_from 
WHERE current = true
GROUP BY imageversionnumber
ORDER BY count DESC

##[5] Which parameters are the most used in RUN Instructions ?
SELECT run_params, count(run_params)
FROM run_params
GROUP BY run_params
ORDER BY count DESC

##[5] Top RUN Instructions
SELECT executable,count(executable), run_params
FROM df_run df NATURAL JOIN run_params rp
WHERE df.current=true
GROUP BY executable, run_params
ORDER BY count(executable) DESC


########################################################################################################################################################
##2. Churn and Co-Evolution
########################################################################################################################################################
##[1] How many times do Dockerfile change (average)
SELECT avg(count)
FROM (
SELECT dock_id, count(*)
FROM dockerfile NATURAL JOIN snapshot 
GROUP BY dock_id) s

##[2] How many times do dockerfiles change with other files?
SELECT avg(count)
FROM(
SELECT dock_id, count(dock_id)
FROM(SELECT s.snap_id, s.dock_id, count(s.snap_id)
FROM snapshot s NATURAL JOIN changed_files c
WHERE c.range_index = 0
GROUP BY s.snap_id
ORDER BY count ASC) g NATURAL JOIN dockerfile
WHERE g.count = 1
GROUP BY dock_id) f

SELECT avg(count)
FROM(
SELECT dock_id, count(dock_id)
FROM(SELECT s.snap_id, s.dock_id, count(s.snap_id)
FROM snapshot s NATURAL JOIN changed_files c
WHERE c.range_index = 0
GROUP BY s.snap_id
ORDER BY count ASC) g NATURAL JOIN dockerfile
WHERE g.count > 1
GROUP BY dock_id) f

##[2.1] Which changes are made when a dockerfile changes alone 
SELECT change_type, count(change_type)
FROM(
SELECT s.snap_id, s.dock_id, count(s.snap_id)
FROM snapshot s NATURAL JOIN changed_files c
WHERE c.range_index = 0
GROUP BY s.snap_id
ORDER BY count ASC) f NATURAL JOIN snap_diff NATURAL JOIN  diff NATURAL JOIN diff_type
WHERE diff_state = 'COMMIT_COMMIT'
GROUP BY change_type
ORDER BY count DESC

##[2.1] Which changes are made when a dockerfile changes with other files together
SELECT change_type, count(change_type)
FROM(
SELECT s.snap_id, s.dock_id, count(s.snap_id)
FROM snapshot s NATURAL JOIN changed_files c
WHERE c.range_index > 1
GROUP BY s.snap_id
ORDER BY count ASC) f NATURAL JOIN snap_diff NATURAL JOIN  diff NATURAL JOIN diff_type
WHERE diff_state = 'COMMIT_COMMIT'
GROUP BY change_type
ORDER BY count DESC

##[3] Which files and file type are changed when a dockerfile is changed in a project?
SELECT full_file_name, count(full_file_name)
FROM changed_files
WHERE range_index = 0
GROUP BY full_file_name
ORDER BY count DESC

SELECT file_type, count(file_type)
FROM changed_files
WHERE range_index = 0
GROUP BY file_type
ORDER BY count DESC

##[9] Which files are changed in a range_index?
SELECT full_file_name
FROM changed_files
WHERE range_index = -1
INTERSECT
SELECT full_file_name
FROM changed_files
WHERE range_index = 0
INTERSECT
SELECT full_file_name
FROM changed_files
WHERE range_index = 1
INTERSECT
SELECT full_file_name
FROM changed_files
WHERE range_index = 2

##[10] How many files changes in average together with a dockerfile (index= 0)
SELECT avg(snap_id)
FROM (
SELECT snap_id, count(snap_id)
FROM changed_files
WHERE range_index = 0
GROUP BY snap_id
ORDER BY count(snap_id) DESC ) s

##[11] List of most changed Instructions?
SELECT instruction, count(instruction)
FROM diff_type
WHERE change_type LIKE '%Update%'
GROUP BY instruction
ORDER BY count(instruction) DESC

##[12] How many COmmits per year and per month
SELECT count(*), date_trunc('year', to_timestamp(commit_date)) s
from snapshot
group by date_trunc( 'year', to_timestamp(commit_date) )
ORDER BY s ASC

SELECT count(*), date_trunc('month', to_timestamp(commit_date)) s
from snapshot
group by date_trunc( 'month', to_timestamp(commit_date) )
ORDER BY s ASC

########################################################################################################################################################
##Sonstige
########################################################################################################################################################
##[2] Which Rules are violated according best practices?
SELECT violated_rules, count(violated_rules) 
FROM violated_rules 
GROUP BY violated_rules 
ORDER BY count DESC


##[3] Docker Usage Adoption rate according USERS/ORGANIZATIONS ?
SELECT count(*), date_trunc('year', to_timestamp(first_docker_commit)) s, i_owner_type
FROM dockerfile
group by date_trunc( 'year', to_timestamp(first_docker_commit)), i_owner_type
ORDER BY s  ASC

##[6] Most used words in Comments
SELECT word, count(*)
FROM ( 
  SELECT regexp_split_to_table(comment, '\s') as word
  FROM df_comment
) t
GROUP BY word
ORDER BY count DESC

##[7] Which Instructions are commented a lot?
SELECT instruction, count(instruction)
FROM df_comment NATURAL JOIN snapshot
WHERE index = true AND instruction LIKE '%before%'
GROUP by instruction
ORDER BY count DESC

##[7] Welche Instructions werden am meisten auskommentiert?


##[8] Preferred Source and Destination of ADD and COPY
SELECT source, count(source) c
FROM df_add
WHERE current=true
GROUP BY source
ORDER BY c DESC

SELECT source, count(source) c
FROM df_copy
WHERE current=true
GROUP BY source
ORDER BY c DESC

SELECT destination, count(destination) c
FROM df_copy
WHERE current=true
GROUP BY destination
ORDER BY c DESC

SELECT destination, count(destination) c
FROM df_add
WHERE current=true
GROUP BY destination
ORDER BY c DESC
```SQL




```
### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
