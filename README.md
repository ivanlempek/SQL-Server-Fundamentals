# SQL-Server-Fundamentals
Creating a database from scratch using SQL Server and Azure Data Studio

-----------------------------------------------------------------------
- Com o Azure Data Studio instalado e a conexão com o SQL Server feita, vamos precisar adicionar 3 extensões para podermos prosseguir;
- Na aba de Extensões ( CTRL + SHIFT + X ) iremos instalar: SQL Server Profiler, SQL Server Import e SQL Server Dacpac;
- Com isso iremos conseguir clicar com o direto em 'Databases' e selecionar a opção 'Data-tier Application Wizard';
- Na próxima tela iremos selecionar a opção 'Import Bacpac' e iremos selecionar o nosso arquivo 'sql_fundamentals.bacpac';
- Esse arquivo contém o banco de dados populado que criei usando todos os fundamentos de SQL;

-----------------------------------------------------------------------
- Para criar esse banco e suas tabelas:


CREATE DATABASE [sql-fundamentals]

USE [sql-fundamentals]
GO

CREATE TABLE [Student]
(
    [Id] UNIQUEIDENTIFIER NOT NULL,
    [Name] NVARCHAR(120) NOT NULL,
    [Email] NVARCHAR(180) NOT NULL,
    [Document] NVARCHAR(20) NOT NULL,
    [Phone] NVARCHAR(20) NOT NULL,
    [Birthdate] DATETIME NULL,
    [CreateDate] DATETIME NOT NULL DEFAULT(GETDATE()),
    CONSTRAINT [PK_Student] PRIMARY KEY ([Id])
);
GO

CREATE TABLE  [Author]
(
    [Id] UNIQUEIDENTIFIER NOT NULL,
    [Name] NVARCHAR(80) NOT NULL,
    [Title] NVARCHAR(80) NOT NULL,
    [Image] NVARCHAR(1024) NOT NULL,
    [Bio] NVARCHAR(2000) NOT NULL,
    [Url] NVARCHAR(450) NOT NULL,
    [Email] NVARCHAR(160) NOT NULL,
    [Type] TINYINT NOT NULL,
    CONSTRAINT [PK_Author] PRIMARY KEY ([Id])
);
GO

CREATE TABLE [Career]
(
    [Id] UNIQUEIDENTIFIER NOT NULL,
    [Title] NVARCHAR(160) NOT NULL,
    [Summary] NVARCHAR(2000) NOT NULL,
    [Url] NVARCHAR(1024) NOT NULL,
    [DurationInMinutes] INT NOT NULL,
    [Active] BIT NOT NULL,
    [Featured] BIT NOT NULL,
    [Tags] NVARCHAR(160) NOT NULL,
    CONSTRAINT [PK_Career] PRIMARY KEY ([Id])
);
GO

CREATE TABLE [Category]
(
    [Id] UNIQUEIDENTIFIER NOT NULL,
    [Title] NVARCHAR(160) NOT NULL,
    [Url] NVARCHAR(1024) NOT NULL,
    [Summary] NVARCHAR(2000) NOT NULL,
    [Order] INT NOT NULL,
    [Description] TEXT NOT NULL,
    [Featured] BIT NOT NULL,
    CONSTRAINT [PK_Category] PRIMARY KEY ([Id])
);
GO

CREATE TABLE [Course]
(
    [Id] UNIQUEIDENTIFIER NOT NULL,
    [Tag] NVARCHAR(20) NOT NULL,
    [Title] NVARCHAR(160) NOT NULL,
    [Summary] NVARCHAR(2000) NOT NULL,
    [Url] NVARCHAR(1024) NOT NULL,
    [Level] TINYINT NOT NULL,
    [DurationInMinutes] INT NOT NULL,
    [CreateDate] DATETIME NOT NULL DEFAULT(GETDATE()),
    [LasUpdateDate] DATETIME NOT NULL,
    [Active] BIT NOT NULL,
    [Free] BIT NOT NULL,
    [Featured] BIT NOT NULL,
    [AuthorId] UNIQUEIDENTIFIER NOT NULL,
    [CategoryId] UNIQUEIDENTIFIER NOT NULL,
    [Tags] NVARCHAR(160) NOT NULL,
    CONSTRAINT [PK_Course] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Course_Author_AuthorId] FOREIGN KEY ([AuthorId]) REFERENCES [Author] ([Id]) ON DELETE NO ACTION,
    CONSTRAINT [FK_Course_Category_CategoryId] FOREIGN KEY ([CategoryId]) REFERENCES [Category] ([Id]) ON DELETE NO ACTION,
);
GO

CREATE TABLE [CareerItem]
(
    [CareerId] UNIQUEIDENTIFIER NOT NULL,
    [CourseId] UNIQUEIDENTIFIER NOT NULL,
    [Title] NVARCHAR(160) NOT NULL,
    [Description] TEXT NOT NULL,
    [Order] TINYINT NOT NULL,
    CONSTRAINT [PK_CareerItem] PRIMARY KEY ([CourseId], [CareerId]),
    CONSTRAINT [FK_CareerItem_Career_CareerId] FOREIGN KEY ([CareerId]) REFERENCES [Career] ([Id]),
    CONSTRAINT [FK_CareerItem_Course_CourseId] FOREIGN KEY ([CourseId]) REFERENCES [Course] ([Id]) 
);
GO

CREATE TABLE [StudentCourse]
(
    [CourseId] UNIQUEIDENTIFIER NOT NULL,
    [StudentId] UNIQUEIDENTIFIER NOT NULL,
    [Progress] TINYINT NOT NULL,
    [Favorite] BIT NOT NULL,  
    [StartDate] DATETIME NOT NULL,
    [LastUpdateDate] DATETIME NULL,
    CONSTRAINT [PK_StudentCourse] PRIMARY KEY ([CourseId], [StudentId]),
    CONSTRAINT [FK_StudentCourse_Course_CourseId] FOREIGN KEY ([CourseId]) REFERENCES [Course] ([Id]),
    CONSTRAINT [FK_StudentCourse_Student_StudentId] FOREIGN KEY ([StudentId]) REFERENCES [Student] ([Id])

)
      
-----------------------------------------------------------------------
- E para poder fazer os filtros de pesquisa utilizando esse banco:

	SELECT * FROM [Author]
  SELECT * FROM [Course]
	SELECT * FROM [Career]
	SELECT * FROM [CareerItem]
	SELECT * FROM [Student]
	SELECT * FROM [StudentCourse]
	
---------------------------
- Select de cursos pelas colunas Id,Tag,Title,Url e Summary por ordem de data de criação e que estão ativos (podem ser visualizados):

SELECT
    [Id],
    [Tag],
    [Title],
    [Url],
    [Summary]
FROM
    [Course]
WHERE
    [Active] = 1
ORDER BY
    [CreateDate] DESC

---------------------------
- Fazendo um INNER JOIN de Cursos com Categoria e Autor (Para que a gente possa visualizar o titulo das categorias e os nomes dos autores junto dos cursos) e adicionando um ALIAS no titulo das categorias e nos nomes dos autores:

SELECT
    [Course].[Id],
    [Course].[Tag],
    [Course].[Title],
    [Course].[Url],
    [Course].[Summary],
    [Category].[Title] AS [Category],
    [Author].[Name]
FROM
    [Course]
    INNER JOIN [Category] ON [Course].[CategoryId] = [Category].[Id] 
    INNER JOIN [Author] ON [Course].[AuthorId] = [Author].[Id]
WHERE
    [Active] = 1
ORDER BY
    [CreateDate] DESC

---------------------------
- Criando uma view desta querie:

CREATE OR ALTER VIEW vwCourses AS
    SELECT
        [Course].[Id],
        [Course].[Tag],
        [Course].[Title],
        [Course].[Url],
        [Course].[Summary],
        [Course].[CreateDate],
        [Category].[Title] AS [Category],
        [Author].[Name] AS [Author]
    FROM
        [Course]
        INNER JOIN [Category] ON [Course].[CategoryId] = [Category].[Id] 
        INNER JOIN [Author] ON [Course].[AuthorId] = [Author].[Id]
    WHERE
        [Active] = 1

---------------------------
- Executando a view em uma nova querie:

SELECT * FROM vwCourses ORDER BY [CreateDate]

---------------------------
- Fazendo o SELECT nas carreiras e adicionando um COUNT  de quantos cursos cada carreira possui :

SELECT
    [Career].[Id],
    [Career].[Title],
    [Career].[Url],
    COUNT([Id]) AS [Courses]
FROM
    [Career]
    INNER JOIN [CareerItem] ON [CareerItem].[CareerId] = [Career].[Id]
GROUP BY
    [Career].[Id],
    [Career].[Title],
    [Career].[Url]

---------------------------
- Inserindo dados nas tabelas de 'Student' e 'StudentCourse':

SELECT * FROM Course
SELECT * FROM [Student]
SELECT * FROM [StudentCourse]


SELECT NEWID() -- Gera um novo Id

INSERT INTO 
    [Student]
VALUES (
    '2606fcd2-27d8-405a-8d39-7e4dddf4d775',
    'Ivan Lempek',
    'ivanlempek@hotmail.com',
    '12345678123',
    '5565694512',
    '1995-06-01',
    GETDATE()
)

INSERT INTO
    [StudentCourse]
VALUES (
    '5d8cf396-e717-9a02-2443-021b00000000',
    '2606fcd2-27d8-405a-8d39-7e4dddf4d775',
    50,
    0,
    '2022-02-02 12:52:41',
    GETDATE()
)

-------------------------
- Visualizando o progresso do aluno;
- Trazendo todos os cursos pelo Id do aluno;
- Visualizando os cursos que o alunos não terminou;
- Visualizando os cursos que o aluno iniciou;
- Criando e executando uma Stored Procedure.


CREATE OR ALTER PROCEDURE spStudentProgress (
    @StudentId UNIQUEIDENTIFIER
)
AS
    SELECT 
    [Student].[Name] AS [Student],
    [Course].[Title] AS [Course],
    [StudentCourse].[Progress]
    FROM
        [StudentCourse]
        INNER JOIN [Student] ON [StudentCourse].[StudentId] = [Student].[Id]
        INNER JOIN [Course] ON [StudentCourse].[CourseId] = [Course].[Id]
    WHERE
        [StudentCourse].[StudentId] = @StudentId
        AND [StudentCourse].[Progress] < 100
        AND [StudentCourse].[Progress] > 0
    ORDER BY
        [StudentCourse].[LastUpdateDate] DESC


---------------------------
- Executando a Procedure:

EXEC spStudentProgress '2606fcd2-27d8-405a-8d39-7e4dddf4d775'
