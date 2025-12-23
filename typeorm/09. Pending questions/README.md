# TypeORM Interview Questions












## **Performance & Optimization**

116. How do you optimize TypeORM queries for performance?
117. What is query caching and how do you implement it?
118. How do you use select to fetch specific columns?
119. What is N+1 query problem and how do you solve it in TypeORM?
120. How do you use relation loading strategies (eager vs lazy)?
121. What is the difference between leftJoinAndSelect and relation loading?
122. How do you implement database indexes for better performance?
123. How do you use query result caching?
124. What is the purpose of the cache option in find operations?
125. How do you monitor and log TypeORM queries?
126. How do you implement connection pooling?
127. What are the best practices for bulk operations?
128. How do you optimize Many-to-Many relationships?
129. How do you handle large result sets efficiently?
130. What is streaming and how do you use it in TypeORM?

## **Subscribers & Listeners**

131. What are Entity Subscribers in TypeORM?
132. What are Entity Listeners?
133. What is the difference between Subscribers and Listeners?
134. How do you implement a global subscriber?
135. What lifecycle events are available (@BeforeInsert, @AfterInsert, etc.)?
136. How do you use @BeforeUpdate and @AfterUpdate?
137. How do you implement soft delete with subscribers?
138. How do you use subscribers for logging or auditing?
139. Can subscribers access transaction context?
140. What are the best practices for using subscribers?

## **Validation & Decorators**

141. How do you integrate class-validator with TypeORM?
142. How do you validate entities before saving?
143. What is the difference between database constraints and validation?
144. How do you create custom validators?
145. How do you handle validation errors?
146. What are transformer decorators and how do you use them?
147. How do you encrypt sensitive data in columns?
148. How do you implement custom column types?
149. What is the @Generated decorator used for?
150. How do you use @ViewEntity decorator?

## **Relations & Loading**

151. What is the difference between relations and joins?
152. How do you load relations with find()?
153. What is the relations option in FindOptions?
154. How do you perform deep relation loading?
155. What is RelationQueryBuilder?
156. How do you add/remove items in Many-to-Many relationships?
157. How do you count relations without loading them?
158. What is the difference between relation().of() methods?
159. How do you query through multiple relation levels?
160. How do you implement conditional relation loading?

## **Error Handling**

161. How do you handle duplicate key errors?
162. What is QueryFailedError?
163. How do you handle connection errors?
164. How do you implement retry logic for failed queries?
165. How do you handle foreign key constraint violations?
166. What is EntityNotFoundError?
167. How do you create custom error classes?
168. How do you handle transaction errors?
169. How do you implement graceful error recovery?
170. What are the common TypeORM errors and their solutions?

## **Testing**

171. How do you set up TypeORM for testing?
172. How do you use in-memory databases for testing?
173. How do you mock repositories in unit tests?
174. How do you implement integration tests with TypeORM?
175. How do you use test fixtures and factories?
176. How do you reset the database between tests?
177. How do you test migrations?
178. How do you test transactions?
179. What are the best practices for testing with TypeORM?
180. How do you use Jest with TypeORM?

## **Architecture & Best Practices**

181. How do you structure a TypeORM project?
182. What is the repository pattern and how do you implement it?
183. How do you implement the unit of work pattern?
184. How do you separate business logic from data access?
185. How do you implement CQRS with TypeORM?
186. What are the best practices for entity design?
187. How do you handle environment-specific configurations?
188. How do you implement multi-tenancy with TypeORM?
189. How do you version your entities?
190. What are the security best practices with TypeORM?

## **Advanced Features**

191. What are Entity Schemas and how do you use them?
192. How do you use TypeORM with GraphQL?
193. How do you implement polymorphic relationships?
194. What is Single Table Inheritance?
195. What is Class Table Inheritance?
196. How do you implement tree structures (Nested Sets, Closure Table)?
197. What is @Tree decorator and its strategies?
198. How do you use spatial data types?
199. How do you implement full-text search indices?
200. How do you use TypeORM with microservices?

## **NestJS Integration**

201. How do you integrate TypeORM with NestJS?
202. What is @InjectRepository decorator?
203. How do you configure TypeORM in NestJS?
204. How do you use forRoot() and forFeature()?
205. How do you implement custom repositories in NestJS?
206. How do you handle migrations in NestJS?
207. How do you implement database transactions in NestJS controllers?
208. How do you test TypeORM repositories in NestJS?
209. What are the best practices for using TypeORM with NestJS?
210. How do you implement database connection per tenant in NestJS?

## **Real-World Scenarios**

211. How do you implement audit logging for all entities?
212. How do you implement role-based access control at the database level?
213. How do you handle time zones in TypeORM?
214. How do you implement data encryption at rest?
215. How do you implement search functionality with filters and sorting?
216. How do you handle file uploads with database records?
217. How do you implement versioning for entities?
218. How do you handle concurrent updates to the same entity?
219. How do you implement event sourcing with TypeORM?
220. How do you migrate from another ORM to TypeORM?

## **TypeORM CLI**

221. What is TypeORM CLI and how do you install it?
222. How do you initialize a new TypeORM project?
223. What commands are available in TypeORM CLI?
224. How do you generate entities from an existing database?
225. How do you generate migrations from entity changes?
226. How do you create a new migration file?
227. How do you create a new subscriber?
228. How do you use the schema:sync command?
229. How do you use the schema:drop command?
230. What is the difference between CLI and programmatic usage?

## **Troubleshooting**

231. Why are my relations not loading?
232. Why is synchronize not creating my tables?
233. How do you debug TypeORM queries?
234. Why are my migrations not running?
235. How do you fix "Cannot use import statement outside a module" error?
236. Why is my connection pool exhausted?
237. How do you fix "EntityMetadataNotFound" error?
238. Why are circular dependencies causing issues?
239. How do you resolve "Cannot create property on string" error?
240. Why is my query builder returning unexpected results?

## **Comparison & Migration**

241. How does TypeORM compare to Sequelize?
242. How does TypeORM compare to Prisma?
243. What are the advantages of using TypeORM?
244. What are the limitations of TypeORM?
245. When should you use TypeORM vs raw SQL?
246. How do you migrate from TypeORM v0.2 to v0.3?
247. What breaking changes exist between major versions?
248. How do you handle deprecated decorators?
249. What are the differences between TypeORM for MySQL vs PostgreSQL?
250. How do you choose between Active Record and Data Mapper patterns?
