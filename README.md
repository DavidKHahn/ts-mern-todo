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





        
        

