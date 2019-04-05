---
title: Android room 存储复杂数据类型
date: 2019-04-05 10:35:18
tags
---

通常，我们用到数据库会有很多种，这里就不做讨论了，我们只来说说如何用room来存储一些复杂数据结构。


首先看此文章的都假设你已经看过了room的简单用法，如果没有看过，那你可能需要先去看看了。
假设，我们从后台返回一个数据类型是这样的：    

          public class News{
            public long id;
            public String title;
            public String title;
            public List<Tag>tags;
          }

        public class Tag{
          public long id;
          public String name;
      }

这种数据类型就比较很常见了，当然了，你第一反应可能是： 这个哪里复杂了，嗯，说不复杂确实不复杂，说复杂呢，是因为用room的话，确实比较麻烦，说白了，就是如何用 room存储带有lis<Object>类型的数据。ok话不多说，我们直接上代码：

   ######bean 的改造

          @Entity(tableName = "news")
           public class News{
          @PrimaryKey(autoGenerate = true)
            public long id;
            public String title;
            public String title;
           @Ignore
            public List<Tag>tags;
          } 


           @Entity(tableName = "tags")
           public class Tag{
                 @PrimaryKey(autoGenerate = true)
                  public long id;
                  public String name;

                   @ColumnInfo(name = "news_id")
                public long newsid;
            }


然后我们重新添加一个bean：NewsWithTags:
               

            public class NewsWithTags{
    @Embedded
    public News news;

      @Relation(parentColumn = "id",entityColumn = "news_id")
       public List<Tag> tags;
        }


注意看这个bean，没有table注解@Embedded 注解的是news这个bean。
@Relation 中parentColumn对应的是news中的id，entityColumn对应的是tag中的news_id,这个值实际上就等于news中的id。
######Dao
    NewsWithTagDao:

     @Dao
    public interface NewsWithTaDao {
    @Transaction
    @Query("SELECT * FROM news")
    List<NewsWithTag> getAllNews();
}

>其他没啥好说的，添加数据要先添加news，然后根据news 的id添加tag，删除数据，要先根据news的id删除tag表中的数据
      

