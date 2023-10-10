# Лабораторная работа 2

***[TOC]***
1.  [Задание 1](#задание-1)

---

*Вариант*: 16 % 6 == 4

1.  ## Задание 1.

    > | № варианта | Запрос                                                                                                          |
    > |------------|-----------------------------------------------------------------------------------------------------------------|
    > | 4          | Сделайте проверку, все ли преподаватели трудоустроены.                                                          |

    *   **КОД**:

        ```psql
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

    *   **КОД**:

        ```psql
        SELECT s.surname, s.name, f.field_name, fc.mark 
          FROM student AS s, field_comprehension AS fc, field AS f
          WHERE s.student_id = fc.student_id AND f.field_id = fc.field
          AND f.field_name ~ 'Базы данных' AND fc.mark = 5
        ```

    *   **OUTPUT**:

        ![Alt text](image-1.png)
