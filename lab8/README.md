# Лабораторная работа №8 по БД

1.  ## Задание 1

    > Ознакомьтесь с функцией searchBySubject() класса Diary, реализующей поиск
    > введенной дисциплины в списке предметов студента. Модифицируйте функцию поиска
    > так, чтобы можно было искать сразу несколько предметов (предметы вводятся в строку
    > поиска через запятую). В случае, если один или несколько из введенных предметов не
    > существует в базе данных, вывести на экран только те предметы, которые есть в дневнике
    > студента.

    Перепишем функцию `searchBySubject()` следующим образом:

    ```cpp
    void Diary::searchBySubject()
    {
        // Эта функция - демонстрация возможности SQL иньекции.
        // См ЛР. 8 или ChatGPT

        QSqlQuery search_query;
        /*QString query = "select field_name, mark FROM field_comprehension "
                        "LEFT OUTER JOIN Field "
                        "ON Field_comprehension.field = Field.field_id "
                        "where student_id =" + QString::number(current_student_.getUserId()) + " AND field_name = '" + QString(ui->search_line->text()) + "'";
        search_query.exec(query);*/

        QString query{"select field_name, mark FROM field_comprehension "
                "LEFT OUTER JOIN Field "
                "ON Field_comprehension.field = Field.field_id "
                "where student_id = :id AND field_name IN ("};

        query = query + prepareSubjects(ui->search_line->text()) + ")";

        search_query.prepare(query);
        search_query.bindValue(":id", current_student_.getUserId());

        search_query.exec();

        if (search_query.size() == 0) {
            ui->search_line->clear();
            ui->search_line->setPlaceholderText("Предметы не найдены!");
        } else {
            fillDiaryMarks(std::move(search_query));
        }
    }
    ```

    Добавим функцию `trim`:

    ```cpp
    inline std::string trim(std::string str)
    {
        std::string white{" \t\n\r\f\v"};
        str.erase(str.find_last_not_of(white)+1);         //suffixing spaces
        str.erase(0, str.find_first_not_of(white));       //prefixing spaces
        return str;
    }
    ```

    Добавим функцию `prepareSubjects`:

    ```cpp
    QString Diary::prepareSubjects(QString initial_subject_str) {
        std::stringstream ss(initial_subject_str.toStdString());
        std::string token, result;

        while(std::getline(ss, token, ',')) {
            // Remove spaces
            token = trim(token);
            result += "'" + token + "', ";
        }

        // Remove the last comma and space
        result = result.substr(0, result.length()-2);

        return QString::fromStdString(result);
    }
    ```

    В класс `Diary` добавим:

    ```cpp
    private:
      ...
      QString prepareSubjects(QString initial_subject_qstr);
      ...
    ```

    Тестовый студент:

    | Логин  | Пароль     |
    | ------ | ---------- |
    | 838389 | 2003-12-23 |

    ![Alt text](image-1.png)

    Строка запроса (`Философия, Основы информационной безопасности, основы боевой магии`)
    после обработки функцией `searchBySubject()` будет выглядеть следующим образом:

    ```sql
    ...
    where student_id = :id AND field_name IN ('Философия', 'Основы информационной безопасности', 'основы боевой магии')
    ```

2.  ## Задание 2

    > Реализуйте защиту от SQL-инъекции для строки поиска предмета. В случае
    > попытки SQL - инъекции программа должна работать в штатном режиме с возможностью
    > продолжать поиск предметов.

    Изменим фукнцию `prepareSubjects`:

    ```cpp
    QString Diary::prepareSubjects(QString initial_subject_qstr) {
        std::string initial_subject_str{initial_subject_qstr.toStdString()};
        CorrectApostrof(initial_subject_str);
        std::stringstream ss(initial_subject_str);
        std::string token, result;

        while(std::getline(ss, token, ',')) {
            // Remove spaces
            token = trim(token);
            result += "'" + token + "', ";
        }

        // Remove the last comma and space
        result = result.substr(0, result.length()-2);

        return QString::fromStdString(result);
    }
    ```

    И добавим предложенную в лабораторной работе фукнкцию `CorrectApostrof`:

    ```cpp
    void Diary::CorrectApostrof(std::string &query)
    {
    size_t pos;
      while ((pos = query.find('\'')) != std::string::npos) {
        query.replace(pos, 1, "`");
      }
    }
    ```

    В класс `Diary` добавим:

    ```cpp
    private:
      ...
      void CorrectApostrof(std::string &query);
      ...
    ```

    Примеры SQL-инъекций, которые ломали БД до введения защиты:
    1.  ```sql
        ') OR 1=1; --'
        ```

        Данная строка в поиске приводила к тому, что отображались все оценки.
    1.  ```sql
        '1' OR 1=1; update field_comprehension set mark = 5 where student_id=856271 ;--'
        ```

        Строка из теории к лабе.

3.  ## Задание 3

    > Добавьте в программу автоматическую блокировку текущей учетной записи в случае
    > попытки выполнить SQL –инъекцию, сопровождаемую сообщением о том, что учетная
    > запись заблокирована (для того, чтобы определить, заблокирован пользователь или нет,
    > добавляется еще одно поле в базе данных пользователя *is_blocked*). Окно с информацией о
    > блокировке должно отображаться поверх всех других окон и содержать кнопку выхода из
    > учетной записи. При попытке зайти в данную учетную запись снова окно о блокировке
    > вновь должно отображаться перед пользователем.
    > Зайдите под любым пользователем и выполните попытку SQL –инъекции.
    > Продемонстрируйте результат.

    Добавим поле *is_blocked* в БД:

    ```sql
    ALTER TABLE users
      ADD is_blocked BOOL DEFAULT FALSE;
    ```

    ```cpp
    void Diary::blockUser() {
        QMessageBox msgBox;
        msgBox.setText("Blocked");
        msgBox.setInformativeText("Ваш аккаунт заблокирован");
        QPushButton *exitButton = msgBox.addButton(QMessageBox::Ok);
        exitButton->setText("Выйти");

        int ret = msgBox.exec();

        if (ret == QMessageBox::Ok) {
          // user clicked OK button
            emit blocked();
        }
    }
    ```

    ```cpp
    void Diary::CorrectApostrof(std::string &query)
    {
    size_t pos;
      while ((pos = query.find('\'')) != std::string::npos) {
        query.replace(pos, 1, "`");
        blockUser();
      }
    }
    ```

    В класс `Diary` добавим:

    ```cpp
    signals:
      void blocked();

    private:
      ...
      void blockUser();
      ...
    ```

    В классе `Student` перенесем поле `diary_` в публичный скоуп.

    В `MainWindow` добавим:

    ```cpp
    private slots:
      void blocked();
    ```

    ```cpp
    MainWindow::MainWindow(QWidget *parent)
        : QMainWindow(parent)
        , ui(new Ui::MainWindow)
    {
      ...
      connect(student_interface_->diary_, &Diary::blocked, this, &MainWindow::blocked);
    }

    ```cpp
    void MainWindow::blocked() {
      stackedWidget->setCurrentWidget(login_form_);
    }
    ```
