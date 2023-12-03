# Лабораторная работа №7 по БД

## Выбор варианта

16 % 5 == 1 (+1)

1.  ## Задание 1

    > Напишите скрипт, в результате работы которого будет выведено ФИО преподавателя,
    > его ставка и реальная занятость – сумма ЗЕТ всех читаемых им дисциплин

    **КОД**:

    ```pgsql
    do
    $$
    DECLARE
      professor record;
    BEGIN
      FOR professor IN
        SELECT p.surname, p.name, p.patronymic, e.wage_rate, SUM(f.zet) AS zet
          FROM professor AS p
          INNER JOIN employment AS e ON p.professor_id = e.professor_id
          INNER JOIN field AS f ON f.professor_id = p.professor_id
          GROUP BY p.professor_id, e.wage_rate
      loop
        raise notice 'ФИО: % % %', professor.surname, professor.name, professor.patronymic;
        raise notice 'ставка: %', professor.wage_rate;
        raise notice 'реальная занятость: %', professor.zet;
		raise notice '';
      end loop;
    END
    $$;
    ```

    **OUTPUT**:

    ```
    NOTICE:  ФИО: Хисамов Василь Тагирович
    NOTICE:  ставка: 1.50
    NOTICE:  реальная занятость: 8
    NOTICE:
    NOTICE:  ФИО: Лазарева Мария Викторовна
    NOTICE:  ставка: 0.60
    NOTICE:  реальная занятость: 12
    NOTICE:
    NOTICE:  ФИО: Сорин Петр Николаевич
    NOTICE:  ставка: 0.25
    NOTICE:  реальная занятость: 5
    ...
    ```
