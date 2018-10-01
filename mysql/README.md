# mysql

``` mysql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `class_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE `class` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

``` go
import "github.com/gpYang/go-databases/mysql"
func main() {
   mysql.LogSQL(func(q string) {
		fmt.Println(q)
	})
	h, _ := mysql.Instance("root:root@tcp(10.10.19.117:3306)/f_center?parseTime=true")
	// nested transaction 1
	h.Begin()
	// h.From("class").
	// Insert([]string{"name"}, map[int]map[string]interface{}{0: map[string]interface{}{"name": "class one"}})
	// INSERT INTO `class` (`name`) VALUE ('class one')
	// h.From("user").
	// Insert([]string{"name", "age", "class_id"}, map[int]map[string]interface{}{0: map[string]interface{}{"name": "Aaron", "age": "18", "class_id": 1}, 1: map[string]interface{}{"name": "Apple"}})
	// INSERT INTO `user` (`name`,`age`,`class_id`) VALUE ('Aaron','18','1'), ('Apple',NULL,NULL)
	fmt.Println(h.
		Field("u.id, u.name uname, u.age, c.name cname").
		From("user", "u").
		Join("class", "u.id = c.id", "c").
		// whereString("uname = Aaron")
		Where("u.name", "=", "Aaron").
		Limit(1).
		Offset(0).
		Order("u.name", "asc").
		// Select()
		// Count()
		Find())
	// fmt.Println(h.Select("SELECT u.id, u.name uname, u.age, c.name cname FROM `user` `u` JOIN `class` `c` ON u.id = c.id WHERE `u`.`name` = 'Aaron' ORDER BY `u`.`name` ASC LIMIT 1"))
	// nested transaction 2
	h.Begin()
	fmt.Println(h.From("user_copy").
		Insert([]string{"name", "age", "class_id"}, map[int]map[string]interface{}{0: map[string]interface{}{"name": "Aaron", "age": "18", "class_id": 1}, 1: map[string]interface{}{"name": "Apple"}}))
	// INSERT INTO `user_copy` (`name`,`age`,`class_id`) VALUE ('Aaron','18','1'), ('Apple',NULL,NULL)
	h.Commit()
	// nested transaction 3
	h.Begin()
	fmt.Println(h.From("user").
		Where("name", "=", "Aaron").
		Update(map[string]interface{}{"age": "19"}))
	h.Rollback()
	h.Commit()
	fmt.Println(h.From("user_copy").
		WhereString("name = 'Aaron'").
		Delete())
	fmt.Println(h.Exec("DELETE FROM user_copy WHERE name = 'Apple'")) 
}
```