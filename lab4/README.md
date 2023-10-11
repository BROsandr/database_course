# Лабораторная работа 4

***[TOC]***
1.  [Задание 1](#задание-1)


---

*Вариант*: 16 % 6 == 4

1.  ## Задание 1.

    > В учебной базе данных одним из допущений является возможность прикрепить
    > только одного преподавателя к дисциплине. Исправьте его.

    *Задача формализованная*: Нужно преобразовать из *1:n* в *m:n*.
    
    Такое преобразование можно сделать, создав связь между таблицами.

    1.  Создадим **связь** *professor_field*, которая связывает *field* и *professor*.

        ```pgsql
        CREATE TABLE professor_field(
          professor_id BIGINT,
          field_id     UUID,
          
          CONSTRAINT professor_fkey
            FOREIGN KEY(professor_id) 
              REFERENCES professor(professor_id)
              ON DELETE CASCADE
        )
        ```

    1.  Установим аттрибуты *professor_id*, *field_id* в качестве **pkey**.

        ```pgsql
        ALTER TABLE professor_field ADD PRIMARY KEY (professor_id, field_id);
        ```

    1.  Копируем данные из таблицы *field* в таблицу *professor_field*.

        ```pgsql
        INSERT INTO professor_field (professor_id, field_id)
          SELECT professor_id, field_id FROM field;
        ```

    1.  Установим аттрибут *field_id* в качестве **fkey**.

        ```pgsql
        ALTER TABLE professor_field ADD FOREIGN KEY (field_id) REFERENCES field(field_id);
        ```

    1.  Удалим аттрибут *professor_id* у таблицы *field*.

        ```pgsql
        ALTER TABLE field
          DROP COLUMN professor_id;
        ```

    Таким образом *ER* диаграмма для таблиц *professor*, *field*, *professor_field*
    выглядит следующим образом:

    ![Alt text](image.png)

2.  ## Задание 2.

    > Добавьте в таблицу Professor поле, содержащее его контактный телефон.
    > Сделайте ограничение, позволяющее хранить номер телефона в формате:
    > +7(XXX)XXX-XX-XX или 8(XXX)XXX-XX-XX

    **КОД**:

    ```pgsql
    ALTER TABLE professor
      ADD mobile_phone varchar(16)
      CONSTRAINT mobile_phone_rus_format
        CHECK (mobile_phone ~ '^(\+7|8)\(\d{3}\)\d{3}-\d{2}-\d{2}$');
    ```

3.  ## Задание 3.

    1.  Добавляем нужно поле в нужную таблицу.

        ```pgsql
        ALTER TABLE students_group
          ADD chieftain BIGINT
        ```

    1.  Делаем нужное поле **fkey**.

        ```pgsql
        ALTER TABLE students_group
          ADD FOREIGN KEY(chieftain)
            REFERENCES student(student_id)
        ```
