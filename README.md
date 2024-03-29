# ts-mern-todo

## SERVER SIDE 
1. `yarn add typescript -g`
2. `yarn add express cors mongoose`
3. `yarn add -D @types/node @types/express @types/mongoose @types/cors`
4. `yarn add -D concurrently nodemon`
5. Update package.json: 

        "scripts": {
            "build": "tsc",
            "start": "concurrently \"tsc -w\" \"nodemon dist/js/app.js\""
          } 
 6. Create a Todo Type:
 
 - types/todo.ts
 
        import { Document } from "mongoose"

        export interface ITodo extends Document {
          name: string
          description: string
          status: boolean
        }
7. Create a Todo Model:

- models/todo.ts

        import { ITodo } from "./../types/todo"
        import { model, Schema } from "mongoose"

        const todoSchema: Schema = new Schema(
          {
            name: {
              type: String,
              required: true,
            },

            description: {
              type: String,
              required: true,
            },

            status: {
              type: Boolean,
              required: true,
            },
          },
          { timestamps: true }
        )

        export default model<ITodo>("Todo", todoSchema)
 8. Create API controllers (Get, Add, Update and Delete Todos)
 
 - controllers/todos/index.ts

        import { Response, Request } from "express"
        import { ITodo } from "./../../types/todo"
        import Todo from "../../models/todo"

        const getTodos = async (req: Request, res: Response): Promise<void> => {
          try {
            const todos: ITodo[] = await Todo.find()
            res.status(200).json({ todos })
          } catch (error) {
            throw error
          }
        }

        const addTodo = async (req: Request, res: Response): Promise<void> => {
          try {
            const body = req.body as Pick<ITodo, "name" | "description" | "status">

            const todo: ITodo = new Todo({
              name: body.name,
              description: body.description,
              status: body.status,
            })

            const newTodo: ITodo = await todo.save()
            const allTodos: ITodo[] = await Todo.find()

            res
              .status(201)
              .json({ message: "Todo added", todo: newTodo, todos: allTodos })
          } catch (error) {
            throw error
          }
        }
        
        const updateTodo = async (req: Request, res: Response): Promise<void> => {
          try {
            const {
              params: { id },
              body,
            } = req
            const updateTodo: ITodo | null = await Todo.findByIdAndUpdate(
              { _id: id },
              body
            )
            const allTodos: ITodo[] = await Todo.find()
            res.status(200).json({
              message: "Todo updated",
              todo: updateTodo,
              todos: allTodos,
            })
          } catch (error) {
            throw error
          }
        }
        
        const deleteTodo = async (req: Request, res: Response): Promise<void> => {
          try {
            const deletedTodo: ITodo | null = await Todo.findByIdAndRemove(
              req.params.id
            )
            const allTodos: ITodo[] = await Todo.find()
            res.status(200).json({
              message: "Todo deleted",
              todo: deletedTodo,
              todos: allTodos,
            })
          } catch (error) {
            throw error
          }
        }

        export { getTodos, addTodo, updateTodo, deleteTodo }
9. Create API routes

 - routes/index.ts

        import { Router } from "express"
        import { getTodos, addTodo, updateTodo, deleteTodo } from "../controllers/todos"

        const router: Router = Router()

        router.get("/todos", getTodos)

        router.post("/add-todo", addTodo)

        router.put("/edit-todo/:id", updateTodo)

        router.delete("/delete-todo/:id", deleteTodo)

        export default router
10. Create a Server
 - nodemon.json

        {
            "env": {
                "MONGO_USER": "your-username",
                "MONGO_PASSWORD": "your-password",
                "MONGO_DB": "your-db-name"
            }
        }
You can get the credentials by creating a new cluster on MongoDB Atlas.

 - app.ts

        import express, { Express } from "express"
        import mongoose from "mongoose"
        import cors from "cors"
        import todoRoutes from "./routes"

        const app: Express = express()

        const PORT: string | number = process.env.PORT || 4000

        app.use(cors())
        app.use(todoRoutes)

        const uri: string = `mongodb+srv://${process.env.MONGO_USER}:${process.env.MONGO_PASSWORD}@clustertodo.raz9g.mongodb.net/${process.env.MONGO_DB}?retryWrites=true&w=majority`
        const options = { useNewUrlParser: true, useUnifiedTopology: true }
        mongoose.set("useFindAndModify", false)

        mongoose
          .connect(uri, options)
          .then(() =>
            app.listen(PORT, () =>
              console.log(`Server running on http://localhost:${PORT}`)
            )
          )
          .catch(error => {
            throw error
          })
          
## CLIENT SIDE
1. `npx create-react-app my-app --template typescript`
2. `yarn add axios`

3. Make sure folder structure looks like this:

        ├── node_modules
        ├── public
        ├── src
        |  ├── API.ts
        |  ├── App.test.tsx
        |  ├── App.tsx
        |  ├── components
        |  |  ├── AddTodo.tsx
        |  |  └── TodoItem.tsx
        |  ├── index.css
        |  ├── index.tsx
        |  ├── react-app-env.d.ts
        |  ├── setupTests.ts
        |  └── type.d.ts
        ├── tsconfig.json
        ├── package.json
        └── yarn.lock

4. Create a Todo Type
- src/type.d.ts

        interface ITodo {
          _id: string
          name: string
          description: string
          status: boolean
          createdAt?: string
          updatedAt?: string
        }

        interface TodoProps {
          todo: ITodo
        }

        type ApiDataType = {
          message: string
          status: string
          todos: ITodo[]
          todo?: ITodo
        }

5. Fetch data from the API
- src/API.ts
        
     **Get:**
        
        import axios, { AxiosResponse } from "axios"

        const baseUrl: string = "http://localhost:4000"

        export const getTodos = async (): Promise<AxiosResponse<ApiDataType>> => {
          try {
            const todos: AxiosResponse<ApiDataType> = await axios.get(
              baseUrl + "/todos"
            )
            return todos
          } catch (error) {
            throw new Error(error)
          }
        }
        
     **Create/Add (POST):**
        
         export const addTodo = async (
          formData: ITodo
        ): Promise<AxiosResponse<ApiDataType>> => {
          try {
            const todo: Omit<ITodo, "_id"> = {
              name: formData.name,
              description: formData.description,
              status: false,
            }
            const saveTodo: AxiosResponse<ApiDataType> = await axios.post(
              baseUrl + "/add-todo",
              todo
            )
            return saveTodo
          } catch (error) {
            throw new Error(error)
          }
        }
        
     **Update (PUT):**
        
        export const updateTodo = async (
          todo: ITodo
        ): Promise<AxiosResponse<ApiDataType>> => {
          try {
            const todoUpdate: Pick<ITodo, "status"> = {
              status: true,
            }
            const updatedTodo: AxiosResponse<ApiDataType> = await axios.put(
              `${baseUrl}/edit-todo/${todo._id}`,
              todoUpdate
            )
            return updatedTodo
          } catch (error) {
            throw new Error(error)
          }
        }
        
     **Delete:**
        
        export const deleteTodo = async (
          _id: string
        ): Promise<AxiosResponse<ApiDataType>> => {
          try {
            const deletedTodo: AxiosResponse<ApiDataType> = await axios.delete(
              `${baseUrl}/delete-todo/${_id}`
            )
            return deletedTodo
          } catch (error) {
            throw new Error(error)
          }
        }
        
6. Create the components
**Add Todo Form**
- components/AddTodo.tsx

        import React, { useState } from 'react'

        type Props = { 
          saveTodo: (e: React.FormEvent, formData: ITodo | any) => void 
        }

        const AddTodo: React.FC<Props> = ({ saveTodo }) => {
          const [formData, setFormData] = useState<ITodo | {}>()

          const handleForm = (e: React.FormEvent<HTMLInputElement>): void => {
            setFormData({
              ...formData,
              [e.currentTarget.id]: e.currentTarget.value,
            })
          }

          return (
            <form className='Form' onSubmit={(e) => saveTodo(e, formData)}>
              <div>
                <div>
                  <label htmlFor='name'>Name</label>
                  <input onChange={handleForm} type='text' id='name' />
                </div>
                <div>
                  <label htmlFor='description'>Description</label>
                  <input onChange={handleForm} type='text' id='description' />
                </div>
              </div>
              <button disabled={formData === undefined ? true: false} >Add Todo</button>
            </form>
          )
        }

        export default AddTodo


**Display a Todo**
- components/TodoItem.tsx

        import React from "react"

        type Props = TodoProps & {
          updateTodo: (todo: ITodo) => void
          deleteTodo: (_id: string) => void
        }

        const Todo: React.FC<Props> = ({ todo, updateTodo, deleteTodo }) => {
          const checkTodo: string = todo.status ? `line-through` : ""
          return (
            <div className="Card">
              <div className="Card--text">
                <h1 className={checkTodo}>{todo.name}</h1>
                <span className={checkTodo}>{todo.description}</span>
              </div>
              <div className="Card--button">
                <button
                  onClick={() => updateTodo(todo)}
                  className={todo.status ? `hide-button` : "Card--button__done"}
                >
                  Complete
                </button>
                <button
                  onClick={() => deleteTodo(todo._id)}
                  className="Card--button__delete"
                >
                  Delete
                </button>
              </div>
            </div>
          )
        }

        export default Todo
        
7. Fetch and Display data
- App.tsx

        import React, { useEffect, useState } from 'react'
        import TodoItem from './components/TodoItem'
        import AddTodo from './components/AddTodo'
        import { getTodos, addTodo, updateTodo, deleteTodo } from './API'

        const App: React.FC = () => {
          const [todos, setTodos] = useState<ITodo[]>([])

          useEffect(() => {
            fetchTodos()
          }, [])

          const fetchTodos = (): void => {
            getTodos()
            .then(({ data: { todos } }: ITodo[] | any) => setTodos(todos))
            .catch((err: Error) => console.log(err))
          }


        const handleSaveTodo = (e: React.FormEvent, formData: ITodo): void => {
          e.preventDefault()
          addTodo(formData)
            .then(({ status, data }) => {
              if (status !== 201) {
                throw new Error("Error! Todo not saved")
              }
              setTodos(data.todos)
            })
            .catch(err => console.log(err))
        }


        const handleUpdateTodo = (todo: ITodo): void => {
          updateTodo(todo)
            .then(({ status, data }) => {
              if (status !== 200) {
                throw new Error("Error! Todo not updated")
              }
              setTodos(data.todos)
            })
            .catch(err => console.log(err))
        }

        const handleDeleteTodo = (_id: string): void => {
          deleteTodo(_id)
            .then(({ status, data }) => {
              if (status !== 200) {
                throw new Error("Error! Todo not deleted")
              }
              setTodos(data.todos)
            })
            .catch(err => console.log(err))
        }


          return (
            <main className='App'>
              <h1>My Todos</h1>
              <AddTodo saveTodo={handleSaveTodo} />
              {todos.map((todo: ITodo) => (
                <TodoItem
                  key={todo._id}
                  updateTodo={handleUpdateTodo}
                  deleteTodo={handleDeleteTodo}
                  todo={todo}
                />
              ))}
            </main>
          )
        }

        export default App







        
        

