-- Lấy thông tin các bộ phim gồm: tiêu đề, mô tả, tên đạo diễn, tên biên kịch (bảng writers - trả về dạng array),
-- độ dài, rating, của các bộ phim thuộc loại ‘Movie’.

CREATE OR REPLACE VIEW movie_view as
select m.id , m.title, m.description, d.full_name as director,
JSON_ARRAYAGG(w.full_name) as writers,
m.`length` as length, m.rating as rating
FROM movie m
LEFT JOIN director d on d.id = m.id_director
LEFT JOIN movie_writers mw on m.id = mw.id_movie 
LEFT JOIN writers w on w.id = mw.id_writer 
GROUP BY m.id 

SELECT *
FROM movie_view

-- Liệt kê các hãng phim (Bảng manufacturer) và số lượng phim thuộc hãng đó

select m.name as manufacturer , COUNT(m.id) as quantity_movie 
from manufacturer m  inner join movie_manufacturer mm 
on m.id = mm.id_manufacturer  
INNER JOIN movie m2  
on m2.id = mm.id_movie  
GROUP BY m.id 

-- Liệt kê các phim thuộc loại TV Series đã hoàn thành (current_episode = episode)

SELECT m.title
FROM movie m inner join title_type tt 
on m.id_title_type = tt.id
WHERE m.current_episode = m.episode 
AND tt.name = 'TV Series'

-- Liệt kê tên phim và trailer có trạng thái active của phim đó
select m.title, t.link as trailer
FROM movie m inner join movie_trailer mt 
on m.id = mt.id_movie
INNER JOIN trailer t 
on t.id = mt.id_trailer
WHERE mt.status = 'active'


-- Liệt kê tiêu đê, mô tả, poster, độ dài và điểm imdb của các phim thuộc loại Movie và sắp xếp theo điểm imdb giảm dần

SELECT m.title, m.description, m.`length`, m.imdb
FROM movie m inner join title_type tt 
on m.id_title_type = tt.id 
WHERE tt.id = 1
order by m.imdb DESC 

-- Liệt kê tiêu đề, mô tả, poster, độ dài, thể loại (bảng genres - trả về dữ liệu dạng array), 
-- số tập và số tập đã công chiếu, của các phim thuộc loại TV mini Series, sắp xếp theo ngày công chiếu mới nhất

CREATE OR REPLACE VIEW movie_view6 as
select m.title, m.description, m.poster, m.`length`, JSON_ARRAYAGG(g.name) as genres, m.episode, m.current_episode, m.release_date 
FROM movie m 
left join movie_genres mg on m.id = mg.id_movie 
LEFT JOIN genres g on g.id = mg.id_genres 
inner JOIN title_type tt                                                                        
on m.id_title_type = tt.id 
WHERE tt.id = 3
group by m.id 
ORDER BY m.release_date DESC  

SELECT *
FROM movie_view6

-- Liệt kê tiêu đề, mô tả, đạo diễn, biên kịch (array), poster, độ dài, thể loại (bảng genres - trả về dữ liệu dạng array), 
-- tên diễn viên (array) của các phim thuộc loại Movie của 10 bộ phim có điểm imdb cao nhất

create or REPLACE VIEW movie_view7 as
select m.title, m.description, d.full_name as director, JSON_ARRAYAGG(w.full_name) as writers,
m.poster, m.`length`, JSON_ARRAYAGG(g.name) as genres, JSON_ARRAYAGG(a.full_name) as actors
FROM movie m
LEFT JOIN director d on d.id = m.id_director
LEFT JOIN movie_writers mw on m.id = mw.id_movie 
LEFT JOIN writers w on w.id = mw.id_writer 
LEFT JOIN movie_genres mg on m.id = mg.id_movie 
LEFT JOIN genres g on g.id = mg.id_genres
LEFT JOIN movie_actor ma on m.id = ma.id_movie
LEFT JOIN actor a on a.id = ma.id_actor
INNER JOIN title_type tt on m.id_title_type = tt.id
WHERE tt.id = 1
GROUP BY m.id 
ORDER BY m.imdb DESC 
LIMIT 10

SELECT *
FROM movie_view7