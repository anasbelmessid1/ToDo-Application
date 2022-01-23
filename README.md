# Todo-App

*<h2>Table of contents</h2>*

   [Introduction](#Introduction)
  
   [Functionality](#Functionality)
   
   [Conclusion](#Conclusion)
   

*<h2>Introduction</h2>*

In this practical course we are going to create an application in QT Designer by implementing The Model-View-Controller (MVC) approach which is basically an architectural pattern that will separate our application into three main logical components: the model, the view, and the controller. Each of these components are built to handle specific development aspects of an application. MVC is one of the most frequently used industry-standard web development framework to create scalable and extensible projects. The application we wil be working on is a ToDoApp which is an application that manages and stores an archive of al the pending and finished tasks . It includes all the features of a main appilcation : menues , actions and toolbar .

* QSqlQueryModel :
To create this applicaton , we chose to work with QSqlQueryModel which is The QSqlQueryModel class provides a read-only data model for SQL result sets.
SqlQueryModel is a high-level interface for executing SQL statements and traversing the result set. It is built on top of the lower-level QSqlQuery and can be used to provide data to view classes such as QTableView.

* QTableView :

A QTableView implements a table view that displays items from a model. This class is used to provide standard tables that were previously provided by the QTable class, but using the more flexible approach provided by Qt's model/view architecture.
Enough talking and let's hop in to our application and see what it consists of  . Our application contains a menubar at the top that is composed of File , options , and Help . Each one of these has its own components as shown bellow :


The application contains also a ToolBar that is composed of shortcuts of some functionalities : 


The application also contains a main window that shows us infinformation about  on today ,pending and finished tasks. 
When you click on the button of adding a task a dialog is displayed and it is composed of the Decription label: where we describe our tasks,  The tag : choosing between Life,Work and other ,
The due date: sets  the date of the task
Checkbox : cheked or not depending on the task if its finihsed.




*<h2>Functionality</h2>*

*<h3>Adding a task</h3>*

```javascript
QDate Task::getDate(){              
    return ui->dateEdit->date();
}


void Task::setDesc(QString desc){
    ui->description->setText(desc);
}

void Task::setTag(QString currText){
    ui->Tag->setCurrentText(currText);
}

void Task::setDate(QString str){

    //Split the str according to "/" char
    auto dateList = str.split("/");

    //Construire la date
    if(!dateList.isEmpty())
    {
        auto date = QDate(dateList.at(2).toInt(), dateList.at(1).toInt(), dateList.at(0).toInt());
        ui->dateEdit->setDate(date);

```
And here is the implementation of the added slot 

```javascript
void todoApp::on_actionNew_Task_triggered()
{
    //create the dialog
    Task task;

    //Execute the dialog
    auto reply = task.exec();

    //if the the dialog is accepted, show the task in the appropriate table view
    if(reply == Task::Accepted)
    {
        addEnty(task.getDesc(), task.getDate(), task.getTag(), task.getIfIsFinished());

        if(task.getDate() == QDate::currentDate())
        {
            //define the query on the database 
            auto  query = QSqlQuery(db);
                        QString view{"SELECT * FROM todayTasks"};
            query.exec(view);
            todayTaskModel->setQuery(query);
            ui->todayTask->setModel(todayTaskModel);
        }
        else if(task.getDate() > QDate::currentDate())
        {
            //define the query on the database
            auto  query = QSqlQuery(db);
            QString view{"SELECT * FROM pendingTasks"};
            query.exec(view);
            pendingTaskModel->setQuery(query);
            ui->pendingTask->setModel(pendingTaskModel);
        }
        else
        {
            //define the query on the database
            auto  query = QSqlQuery(db);
            QString view{"SELECT * FROM finishedTasks"};
            query.exec(view);
            finishedTaskModel->setQuery(query);
            ui->finishedTask->setModel(finishedTaskModel);
        }

    }


```

*<h3>View pending  task</h3>*

To view a specific task we use setVisible(bool) method

```javascript
void todoApp::on_actionPending_Task_triggered()
{
    if(ui->pendingTask->isVisible() && ui->pendingLabel->isVisible())
    {
        ui->pendingTask->setVisible(false);
        ui->pendingLabel->setVisible(false);
    }
    else
    {
        ui->pendingTask->setVisible(true);
        ui->pendingLabel->setVisible(true);
    }
}
```

*<h3>View today task</h3>*



```javascript
void todoApp::on_actionToday_Task_triggered()
{
    if(ui->todayTask->isVisible() && ui->todayLabel->isVisible())
    {
        ui->todayTask->setVisible(false);
        ui->todayLabel->setVisible(false);
    }
    else
    {
        ui->todayTask->setVisible(true);
        ui->todayLabel->setVisible(true);
    }
}
```
*<h3>View finished task</h3>* 

```javascript


void todoApp::on_actionFinished_Tasks_triggered()
{
    if(ui->finishedTask->isVisible() && ui->finishedLabel->isVisible())
    {
        ui->finishedTask->setVisible(false);
        ui->finishedLabel->setVisible(false);
    }
    else
    {
        ui->finishedTask->setVisible(true);
        ui->finishedLabel->setVisible(true);
    }
}

```

*<h3>Removing task</h3>* 

```javascript
void todoApp::on_actionDelete_Tasks_triggered()
{
    //Get the selected rows in todayTasks
    QModelIndexList selectToday = ui->todayTask->selectionModel()->selectedRows();

    //Remove the selected rows from todayTaskModel
    for(int i=0; i<selectToday.count(); i++)
    {
        //Define a query on the DB
        auto query = QSqlQuery(db);

        //Define the body of the query
        QString del{"DELETE FROM todayTasks WHERE description="
                    "'"+selectToday[i].data(Qt::EditRole).toString()+"'"};

        //Execute the query
        if(!query.exec(del))
            qDebug() << "Cannot delete row!";

        //set the query to the model
        todayTaskModel->setQuery(query);

        //submit changes
        todayTaskModel->submit();

        //show changes
        auto todayQuery = QSqlQuery(db);
        QString today{"SELECT * FROM todayTasks"};
        todayQuery.exec(today);
        todayTaskModel->setQuery(todayQuery);
        ui->todayTask->setModel(todayTaskModel);

    }

    //Get the selected rows in todayTasks
    QModelIndexList selectPending = ui->pendingTask->selectionModel()->selectedRows();

    //Remove the selected rows from todayTaskModel
    for(int i=0; i<selectPending.count(); i++)
    {
        //Define a query on the DB
        auto query = QSqlQuery(db);

        //Define the body of the query
        QString del{"DELETE FROM pendingTasks WHERE description="
                    "'"+selectPending[i].data(Qt::EditRole).toString()+"'"};

        //Execute the query
        if(!query.exec(del))
            qDebug() << "Cannot delete row!";

        //set the query to the model
        pendingTaskModel->setQuery(query);

        //submit changes
        pendingTaskModel->submit();

        //show changes
        auto pendingQuery = QSqlQuery(db);
        QString pending{"SELECT * FROM pendingTasks"};
        pendingQuery.exec(pending);
        pendingTaskModel->setQuery(pendingQuery);
        ui->pendingTask->setModel(pendingTaskModel);

    }

    //Get the selected rows in todayTasks
    QModelIndexList selectFinished = ui->finishedTask->selectionModel()->selectedRows();

    //Remove the selected rows from todayTaskModel
    for(int i=0; i<selectFinished.count(); i++)
    {
        //Define a query on the DB
        auto query = QSqlQuery(db);

        //Define the body of the query
        QString del{"DELETE FROM finishedTasks WHERE description="
                    "'"+selectFinished[i].data(Qt::EditRole).toString()+"'"};

        //Execute the query
        if(!query.exec(del))
            qDebug() << "Cannot delete row!";

        //set the query to the model
        finishedTaskModel->setQuery(query);

        //submit changes
        finishedTaskModel->submit();

        //show changes
        auto finishedQuery = QSqlQuery(db);
        QString finished{"SELECT * FROM finishedTasks"};
        finishedQuery.exec(finished);
        finishedTaskModel->setQuery(finishedQuery);
        ui->finishedTask->setModel(finishedTaskModel);

    }
}
```

*<h3>Exit the application </h3>* 

```javascript
void todoApp::on_action_Exit_triggered()
{
    this->close();
}
```
*<h3>Edit the application: </h3>* 
```javascript
void todoApp::editTaskSlot()
{
    auto currRow = ui->todayTask->currentIndex().row();

    qDebug() << currRow;

    Task T;
    auto reply = T.exec();

    if(reply == Task::Accepted)
    {
        auto query = QSqlQuery(db);

        //Ajouter les champs du dialogue à la base de données
        QString addInfo{"UPDATE todayTasks (description, date, tag, finished)"
                        " VALUES('%1','%2', '%3', '%4') WHERE id = "};

//        //Executer la requete
//        if(!query.exec(addInfo.arg(desc).arg(date.toString()).arg(tag).arg(finished)))
//            QMessageBox::critical(this,"Info","Cannot add the Entry");

        //Connecter le modèle à la requete
        todayTaskModel->setQuery(query);
    }
```
We add this code in the constructor in order to find the tasks that we already added 

```javascript
void todoApp::showDB()
{
    //Show today tasks
        auto todayQuery = QSqlQuery(db);
        QString today{"SELECT * FROM todayTasks"};
        todayQuery.exec(today);
        todayTaskModel->setQuery(todayQuery);
        ui->todayTask->setModel(todayTaskModel);

    //Show pending tasks
         auto pendingQuery = QSqlQuery(db);
         QString pending{"SELECT * FROM pendingTasks"};
         pendingQuery.exec(pending);
         pendingTaskModel->setQuery(pendingQuery);
         ui->pendingTask->setModel(pendingTaskModel);



    //Show finished tasks
        auto finishedQuery = QSqlQuery(db);
        QString finished{"SELECT * FROM finishedTasks"};
        finishedQuery.exec(finished);
        finishedTaskModel->setQuery(finishedQuery);
        ui->finishedTask->setModel(finishedTaskModel);

}
```
