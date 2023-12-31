# Лабораторная работа 3

***[TOC]***
1.  [Задание 1](#задание-1)
2.  [Задание 2](#задание-2)
3.  [Задание 3](#задание-3)
4.  [Задание 4](#задание-4)
5.  [Задание 5](#задание-5)

---

*Вариант*: 16 % 6 == 4

1.  ## Задание 1.

    > | № варианта | Запрос                                                                                                          |
    > |------------|-----------------------------------------------------------------------------------------------------------------|
    > | 4          | Сделайте проверку, все ли преподаватели трудоустроены.                                                          |

    *   **КОД**:

        ```pgsql
        SELECT p.name, p.surname, p.patronymic, e.structural_unit_number
          FROM professor AS p
          LEFT OUTER JOIN employment AS e
          ON p.professor_id = e.professor_id
          WHERE e.structural_unit_number IS null
        ```

    *   **OUTPUT**:

        ![Alt text](image.png)

    *   **ОТВЕТ**: Не все преподаватели трудоустроены.
2.  ## Задание 2.

    > Выведите Фамилии и Имена студентов, кто получил 5 по Базам данных.

    ~~Попытка 1:~~

    *   **КОД**:

        ```pgsql
        SELECT s.surname, s.name, f.field_name, fc.mark 
          FROM student AS s, field_comprehension AS fc, field AS f
          WHERE s.student_id = fc.student_id AND f.field_id = fc.field
          AND f.field_name ~ 'Базы данных' AND fc.mark = 5
        ```

    *   **OUTPUT**:

        ![Alt text](image-1.png)

    Попытка 2:

    *   **КОД**:

        ```pgsql
        SELECT s.surname, s.name, f.field_name, fc.mark 
          FROM student AS s
          INNER JOIN field_comprehension AS fc
          ON s.student_id = fc.student_id
          INNER JOIN field AS f
          ON f.field_id = fc.field
          WHERE f.field_name ~ 'Базы данных' AND fc.mark = 5
        ```

    *   **OUTPUT**:

        ![Alt text](image-3.png)

3.  ## Задание 3.

    > Выведите студента, его группу и преподавателя, у которого фамилия = фамилия студента.

    *   **КОД**:

        ```pgsql
        SELECT s.surname AS student_surname, s.students_group_number, p.surname AS professor_surname
          FROM student AS s
          INNER JOIN professor AS p
          ON s.surname = p.surname
        ```

    *   **OUTPUT**:

        ![Alt text](image-4.png)

4.  ## Задание 4.

    > Выведите фамилию и имя всех студентов, кто учится заочно.

    *   **КОД**:

        ```pgsql
        SELECT s.surname, s.name, g.enrolment_status
          FROM student AS s
          INNER JOIN students_group AS g
          ON s.students_group_number = g.students_group_number
          WHERE g.enrolment_status ~ 'Заочная'
        ```

    *   **OUTPUT**:

        ![Alt text](image-5.png)

5.  ## Задание 5.

    > ![Alt text](image-6.png)

    1.  > INNER JOIN

        > Вывести **Фамилию** и **email** всех должников

        *   **КОД**:

            ```pgsql
            SELECT s.surname, s.email
              FROM student AS s
              INNER JOIN debtor_students AS d
              ON s.surname = d.surname AND s.name = d.name AND s.patronymic = d.patronymic
            ```

        *   **OUTPUT**:

            ![Alt text](image-7.png)

    2.  > INNER JOIN

        > Вывести **Фамилию** преподавателя,
        > который ставит больше всех оценок "3" за какой-либо предмет.

        *   **КОД**:

            ```pgsql
            SELECT p.surname, count(*) as number_of_3
              FROM professor AS p
              INNER JOIN field AS f
              ON p.professor_id = f.professor_id
              INNER JOIN field_comprehension AS fc
              ON fc.field = f.field_id
              WHERE fc.mark = 3
              GROUP BY p.professor_id
              ORDER BY number_of_3 DESC
              LIMIT 1
            ```

        *   **OUTPUT**:

            ![Alt text](image-8.png)

    3.  > RIGHT OUTER JOIN

        > Вывести название предметов, по которым есть задолженности.

        *   **КОД**:

            ```pgsql
            SELECT f.field_name, count(*) AS number_of_debtors
              FROM field AS f
              RIGHT OUTER JOIN debtor_students AS d
              ON f.field_id = d.debt_subject_id::uuid
              GROUP BY f.field_id
              ORDER BY number_of_debtors DESC
            ```

        *   **OUTPUT**:

            ![Alt text](image-9.png)

      4.  > UNION

          > Вывести студентов групп *ИВТ-42* и *ИВТ-43*

          *   **КОД**:

              ```pgsql
              SELECT s.surname, s.students_group_number
                FROM student AS s
                WHERE students_group_number = 'ИВТ-42'

              UNION

              SELECT s.surname, s.students_group_number
                FROM student AS s
                WHERE students_group_number = 'ИВТ-43'
              ```

              *Замечание*: Это не единственный способ решения задачи.
          *   **OUTPUT**:

              ![Alt text](image-10.png)

      5.  > INTERSECT

          > Подсчитать количество однофамильцев в отношении преподаватель-студент.

          *   **КОД**:

              ```pgsql
              SELECT count(*)
                FROM (SELECT s.surname
                        FROM student AS s
                
                      INTERSECT

                      SELECT p.surname
                        FROM professor AS p) AS res;
              ```

          *   **OUTPUT**:

              ![Alt text](image-11.png)
