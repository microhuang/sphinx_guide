sphinx_guide
============

sphinx应用实践技巧

1、exact_hit -- 截止2.2.6，使用短语搜索、相似搜索、近邻搜索，等复杂表达式搜索时，exact_hit不能按预期工作：
    select id,factors() from test where match('word1 word2') option ranker=expr('sum(exact_hit)');    //exact_hit=1
    select id,factors() from test where match('"word1 word2"~10') option ranker=expr('sum(exact_hit)');    //exact_hit=0
    无解决办法。
    
2、phrase_boundary、phrase_boundary_step -- 无法按短语分隔符进行精确匹配。
    解决方法，将记录拆分为多个独立字段索引。 -- sql_query = select id, trim(substring_index(name,'-',1)) as name1, trim(substring_index(substring_index(name,'-',2),'-',-1)) as name2 from test;。注意适当控制字段总数。
    
3、内置评分公式
    SPH_RANK_PROXIMITY_BM25 = sum(lcs*user_weight)*1000+bm25
    SPH_RANK_BM25 = bm25
    SPH_RANK_NONE = 1
    SPH_RANK_WORDCOUNT = sum(hit_count*user_weight)
    SPH_RANK_PROXIMITY = sum(lcs*user_weight)
    SPH_RANK_MATCHANY = sum((word_count+(lcs-1)*max_lcs)*user_weight)
    SPH_RANK_FIELDMASK = field_mask
    SPH_RANK_SPH04 = sum((4*lcs+2*(min_hit_pos==1)+exact_hit)*user_weight)*1000+bm25
    
    SphinxClient::SetRankingMode(SPH_RANK_EXPR, SPH_RANK_SPH04)

4、竞价排名
    SphinxClient::SetSortMode(SPH_SORT_EXPR, '(@weight+if(UserKey=='key',UserRank,0)*CustomWeight)');
    
5、短语搜索
    "word1 word2"
    
6、近邻搜索
    "word1 word2"~10
    
7、部分搜索
    "word1 word2"/2
    
8、运行程序实时更新属性，SphinxClient::UpdateAttributes，但不支持程序更新索引实体。

9、多索引合并（主索引+增量索引），确保所有索引源使用同一字符编码（utf8）。
    避免因字符编码导致数据不能搜索到问题。
    
10、api选择
    sphinxapi.php不支持factors() -- 乱码？
    sphinxql支持factors()。
    
更多参考http://sphinxsearch.com/docs/current.html
