---
title: "Insert Large Data Manually To Avoid Timeout"
date: 2021-08-10T08:06:25+06:00
description: "ways to prevent jta timeout when insert/update large data into database"
menu:
  sidebar:
    name: Insert Large Data
    identifier: Insert-Large-Data
    parent : SQL-directory
    weight: 10
hero: images/data.jpg
---
As part of a batch project, I have to insert/update large data (up to 10000 rows) into a MSSQL database. This required inserting into dozens of columns and 10 thousand rows in one query, such complex transactions may encounter an error of `jta transaction unexpectedly rolled back (maybe due to a timeout).` Here's how to handle this exception.

## Basic setting 
It is recommended to set the transaction timeout to your custom value, like adding this above the method. The limit of the timeout number is Integer.MAX_VALUE.
```
@Transactional(rollbackFor = Exception.class, timeout = 60)
```  

Setting the number of rows per commit in jdbc properties is also a way to prevent transaction timeout. The example below determines 100 inserts or updates to be carried out in a single database hit.
```
hibernate.jdbc.batch_size=100
```
However, if none of the above settings makes the transaction go well, we can still manually determines the number of inserts that are sent to the database at one time for execution.  

## Every 100 entities commit once 
```java
    @PersistenceUnit(unitName="YourEntityManagerFactoryBean")
    private EntityManagerFactory entityManagerFactory;

    public void batchSaveAll(Set<Entity> entitySet) {
        int i=0;
        EntityManager em = entityManagerFactory.createEntityManager();
        em.getTransaction().begin();
        Iterator<Entity> iterator = entitySet.iterator();
        while (iterator.hasNext()){
            em.persist(iterator.next());
            if(++i%100==0){
                em.flush();
                em.clear();
                em.getTransaction().commit();
                em.getTransaction().begin();
            }
            iterator.remove();
        }
        em.flush();
        em.clear();
        em.getTransaction().commit();
        em.close();
    }
```

## JUnit Test
```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes= Config.class)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@DataJpaTest
public class Test {

    @Autowired
    Repository repository;

    @Test
    public void testSaveAll(){
        Set<Entity> set = new HashSet<>(1000);
        for(int i=0;i<1000;i++){
            set.add(getEntity());
        }
        repository.batchSaveAll(set);
        List<Entity> list2 = repository.findAll();
        assertEquals(1000,list2.size());
    }

    /** Generate Entity */
    private Entity getEntity(){
        Entity entity = new Entity();
        entity.setId(RandomStringUtils.randomAlphabetic(12));
        entity.setMark1(RandomStringUtils.randomAlphabetic(1));
        entity.setMark2(RandomStringUtils.randomAlphabetic(1));
        entity.setReceiptNo(RandomStringUtils.randomAlphabetic(10));
        return entity;
    }
```

## Insert to a temp table
If inserting large data into temp table is your case, without persist() method, sql statement is also available to do the commit every 100 rows. In case of no entity, I put the column values inside a List.
```java
   @Transactional
    public void insertTempTable(TableBean bean, List<Map<String, String>> records) throws Exception{
        String tableName = bean.getTableName();
        
            EntityManager em = entityManagerFactory.createEntityManager();
            em.getTransaction().begin();

            StringBuilder insertSql = new StringBuilder();
            insertSql.append("INSERT INTO "+ tableName +" (Id,Mark1,Mark2,ReceiptNo) VALUES ");

            for(int i=0; i<size; i++) {
                String Id = records.get(i).get("FirstReceiveNo");
                String Mark1 = records.get(i).get("ExitDate");
                String Mark2 = records.get(i).get("ExitInspectTime");
                String ReceiptNo = records.get(i).get("ExitPort");
                
                insertSql.append(" ('");
                insertSql.append(Id).append("','");;
                insertSql.append(Mark1).append("','");
                insertSql.append(Mark2).append("','");
                insertSql.append(ReceiptNo).append("'),");

                if(i!=0 && i%100==0){
                    //delete the "," symbol to make an end of the statement
                    insertSql.deleteCharAt(insertSql.length()-1); 
                    em.createNativeQuery(insertSql.toString()).executeUpdate();
                    em.flush();
                    em.clear();
                    em.getTransaction().commit();
                    em.getTransaction().begin();
                    //reset the stringBuilder
                    insertSql.setLength(0);
                    insertSql.append("INSERT INTO "+ tableName +" (Id,Mark1,Mark2,ReceiptNo) VALUES ");
                }
            }
            //The rest of the data (not enough for 100) should not be forgotten
            insertSql.deleteCharAt(insertSql.length()-1);
            em.createNativeQuery(insertSql.toString()).executeUpdate();
            em.flush();
            em.clear();
            em.getTransaction().commit();
            em.close();
    }
``` 