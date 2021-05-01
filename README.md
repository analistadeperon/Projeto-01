# Projeto-01
USE [PontoVenda]
GO
CREATE TABLE [dbo].[Produto](
[produtoID] [int] NOT NULL,
[descricao] [varchar](50) NOT NULL,
[precoUnitario] [decimal](12, 2) NOT NULL,
[estocada] [int] NOT NULL,CONSTRAINT [PK__Produto__582517A17F60ED59] PRIMARY KEY CLUSTERED
(
[produtoID] ASC
)
) ON [PRIMARY]
GO
CREATE TABLE [dbo].[Cliente](
[clienteID] [int] NOT NULL,
[nome] [varchar](50) NOT NULL,
[sexo] [char](1) NOT NULL,
[email] [varchar](50) NULL,
CONSTRAINT [PK__Cliente__C2FF24BD03317E3D] PRIMARY KEY CLUSTERED
(
[clienteID] ASC
)
) ON [PRIMARY]
GO
CREATE PROCEDURE [dbo].[ReajustarPrecoDeUmProduto]
@produtoID int,
@percentual decimal (5,2)
AS
UPDATE Produto SET precoUnitario = precoUnitario * (1 + @percentual/100)
 WHERE produtoID=@produtoID
GO
CREATE PROCEDURE [dbo].[ReajustarPrecoDeProdutos]
@percentual decimal (5,2)
AS
UPDATE Produto SET precoUnitario = precoUnitario * (1 + @percentual/100)
GO
CREATE PROCEDURE [dbo].[UpdateProduto]
@produtoID int,
@descricao varchar(50),
@precoUnitario decimal (12,2),
@estocada int
AS
 UPDATE Produto SET produtoID = @produtoID,
 descricao = @descricao,
 precoUnitario = @precoUnitario,
 estocada = @estocada
 WHERE produtoID = @produtoID
GO
CREATE PROCEDURE [dbo].[AtualizarEstoque]
@produtoID int,
@qtdade int
AS
UPDATE Produto Set estocada = estocada - @qtdade WHERE produtoID = @produtoID
GO
CREATE PROCEDURE [dbo].[AtualizarCliente]
 @clienteid int, @nome varchar(50),
 @sexo char(01),
 @email varchar(50)
 AS
 UPDATE Cliente SET clienteID = @clienteID,
 nome = @nome,
 sexo = @sexo,
 email = @email
 where clienteID = @clienteid
GO
CREATE PROCEDURE [dbo].[DeleteProduto]
@produtoID int
AS
 DELETE FROM Produto WHERE produtoID = @produtoID
GO
CREATE PROCEDURE [dbo].[DeletarCliente]
 @clienteID int
 AS
 DELETE FROM Cliente Where clienteID = @clienteID
GO
CREATE PROCEDURE [dbo].[InsertProduto]
@produtoID int,
@descricao varchar(50),
@precoUnitario decimal (12,2),
@estocada int
AS
 INSERT INTO Produto VALUES (@produtoID,@descricao,@precoUnitario,@estocada)
GO
CREATE PROCEDURE [dbo].[InserirCliente]
 @clienteid int,
 @nome varchar(50),
 @sexo char(01),
 @email varchar(50)
 AS
 INSERT INTO Cliente (clienteID,nome,sexo,email)
 values (@clienteid,@nome,@sexo,@email)
GO
CREATE PROCEDURE [dbo].[GetProdutos]
AS
 SELECT * FROM Produto
GO
CREATE PROCEDURE [dbo].[GetProduto]
@produtoID int
AS
 SELECT * FROM Produto WHERE produtoID = @produtoID
GO
CREATE TABLE [dbo].[Pedido](
[pedidoID] [int] NOT NULL,[data] [date] NOT NULL,
[valor] [decimal](12, 2) NULL,
[clienteID] [int] NULL,
CONSTRAINT [PK__Pedido__BAF07AE40F975522] PRIMARY KEY CLUSTERED
(
[pedidoID] ASC
)
) ON [PRIMARY]
GO
CREATE PROCEDURE [dbo].[LocalizarTodosClientes]
 AS
 Select * from Cliente
GO
CREATE PROCEDURE [dbo].[LocalizarClientePorID]
 @clienteID int
 AS
 SELECT * FROM Cliente Where clienteID = @clienteID
GO
CREATE TABLE [dbo].[ItensPedido](
[itemNum] [int] NOT NULL,
[qtdade] [int] NOT NULL,
[precoVenda] [decimal](12, 2) NOT NULL,
[pedidoID] [int] NOT NULL,
[produtoID] [int] NOT NULL,
CONSTRAINT [pk_ItensPedido] PRIMARY KEY CLUSTERED
(
[itemNum] ASC,
[pedidoID] ASC
)
) ON [PRIMARY]
GO
CREATE PROCEDURE [dbo].[GetPedidos]
@clienteID int
AS
SELECT * FROM Pedido WHERE clienteID = @clienteID
GO
CREATE PROCEDURE [dbo].[InsertPedido]
@pedidoID int,
@data date,
@valor decimal(12,2),
@clienteID int
AS
INSERT INTO Pedido VALUES (@pedidoID,@data,@valor,@clienteID)
GO
CREATE PROCEDURE [dbo].[UtimoPedido]
AS
if (Select MAX(PedidoID) FROM Pedido) = null
 return Convert(int, 0)else
 Select CONVERT(int, MAX(PedidoID)) FROM Pedido
GO
CREATE PROCEDURE [dbo].[RealizarVendas]
@pedidoID int,
@clienteID int,
@itemNum int,
@qtdade int,
@produtoID int,
@precoVenda decimal (12,2)
AS
BEGIN
 BEGIN TRANSACTION
 BEGIN TRY
 INSERT INTO ItensPedido VALUES 
(@itemNum,@qtdade,@precoVenda,@pedidoID,@produtoID)
 UPDATE Produto SET estocada=estocada - @qtdade WHERE produtoID=@produtoID 
 COMMIT TRANSACTION
 END TRY
 BEGIN CATCH
 ROLLBACK TRANSACTION
 END CATCH
END
GO
CREATE PROCEDURE [dbo].[InsertItensPedido]
@itemNum int,
@qtade int,
@precoVenda decimal(12,2),
@pedidoID int,
@produtoID int
AS
INSERT INTO ItensPedido VALUES 
(@itemNum,@qtade,@precoVenda,@pedidoID,@produtoID)
GO
CREATE PROCEDURE [dbo].[GetItensPedidoPorPedidoID]
@pedidoID int
AS
SELECT * FROM ItensPedido WHERE pedidoID = @pedidoID
GO
CREATE PROCEDURE [dbo].[GetItensPedido]
AS
SELECT * FROM ItensPedido
GO
ALTER TABLE [dbo].[Pedido] CHECK CONSTRAINT [fk_CliPed]
GO
ALTER TABLE [dbo].[ItensPedido] WITH CHECK ADD CONSTRAINT [fk_ItemProduto] 
FOREIGN KEY([produtoID])
REFERENCES [dbo].[Produto] ([produtoID])
GOALTER TABLE [dbo].[ItensPedido] CHECK CONSTRAINT [fk_ItemProduto]
GO
ALTER TABLE [dbo].[ItensPedido] WITH CHECK ADD CONSTRAINT [fk_PediItem] 
FOREIGN KEY([pedidoID])
REFERENCES [dbo].[Pedido] ([pedidoID])
GO
ALTER TABLE [dbo].[ItensPedido] CHECK CONSTRAINT [fk_PediItem]
GO
