# Failover de Logshipping
Etapas para se fazer um failover de log shipping manualmente e programado.

Utilizei as instâncias na imagem abaixo e a base AdventureWroks2017.

![image](https://user-images.githubusercontent.com/100159466/203073549-1f70ff2e-baef-4a1b-b193-0130e522babe.png)

Filtrei os jobs nas minhas instâncias, como mostra na imagem abaixo:

![image](https://user-images.githubusercontent.com/100159466/203074013-dfdb9158-7051-47cf-83ed-87e581794a10.png)

Deixando somente os jobs de LogShipping, visto que teremos que desativá-los.

### Jobs de Logshipping no servidor primário:

![image](https://user-images.githubusercontent.com/100159466/203074060-8926930b-e5dc-4677-8a34-b6ca7572e33c.png)

### Jobs de Logshipping no servidor secundário:

![image](https://user-images.githubusercontent.com/100159466/203074250-9ee34231-8333-4337-8c5a-db8695b832f6.png)

Clique com o botão direito em cima da base primária e vá até a sessão de Logshipping:

![image](https://user-images.githubusercontent.com/100159466/203075056-7dee8432-f5c3-4a6c-bddd-56020e0fabac.png)

Será aberta uma nova janela, selecione um script, como mostro na imagem abaixo, no meu caso, abri uma nova sesão no SSMS:

![image](https://user-images.githubusercontent.com/100159466/203075404-876a3341-1040-45d1-9e27-3b16215a92a8.png)

Após abrir o script, já pode fechar a janela. Este será o script que utilizarem para configurar o Failover.

![image](https://user-images.githubusercontent.com/100159466/203075770-182f4847-d2b7-4287-b2c4-6d1f53b0312f.png)

Utilize o atalho Ctrl + H para poder alterar as instâncias! E Coloque o nome do seu secundário no lugar do seu primário.

![image](https://user-images.githubusercontent.com/100159466/203076995-d8e3eb6d-7857-4f36-8838-485aae8f1250.png)

Troque tudo o que conter o nome da sua isntância primária para a secundária e repita o mesmo processo quando chegar a parte do secundário (trocar o secundário pelo primário).

![image](https://user-images.githubusercontent.com/100159466/203077831-b8deb419-42a7-4430-8b59-648d0f3b089e.png)

### [Link para verificar como ficou o meu script, após as alterações mencionadas acima.](https://github.com/CloretoJannuzzi/Failover-LogShipping/blob/main/failover.sql)

Delete os jobs de backup, copy e restore, o de alerta não é necessário, apenas desative eles.

Segue abaixo um script para deletar um job de LogShipping, onde você só tera que escrever nome do servidor ou da base:

    
    DECLARE @id_job VARCHAR(80)
    SELECT @id_job = job_id FROM msdb..sysjobs WHERE name LIKE 'LS%AdventureWorks2017%'

    EXEC msdb.dbo.sp_delete_job @job_id = @id_job, @delete_unused_schedule = 1

## Início do Failover

Tudo pronto, agora vamos iniciar o processo para fazer o Failover.

Vamos fazer um backup manualmenete de LOG na base primária e colocá-la em restoring:

    BACKUP LOG AdventureWorks2017 TO DISK '\\DSNB90\AdventureWorks2017\Backups\Backup_log_tail.trn'
    WITH NORECOVERY;

No caminho, você especifica a  paste onde irá colocar os backups e o nome dele com o final ".trn"

Após rodar o script você ficará com a sua base no modo restoring:

![image](https://user-images.githubusercontent.com/100159466/203086252-0aec9e25-5466-4189-a03f-a89efe1cf5fa.png)

Rode este script na MSDB de restore para colocar a base secundária on-line:

    RESTORE LOG AdventureWorks2017 FROM DISK = '\\DSNB90\AdventureWorks2017\Backups\Backup_log_tail.trn'
    WITH RECOVERY;

Como ficará:

![image](https://user-images.githubusercontent.com/100159466/203088187-d103c243-cbb5-41ab-b048-9558b091f512.png)

Após isso vamos para aquele script que configuramos antes:

Ele possui duas etapas, uma para ser executada no servidor que será o primário e a outra parte no secundário.

### Antes de Executar o código, verifique os diretórios, para verificar se está no caminho correto.

Roda a primeira parte no novo servidor primário:

![image](https://user-images.githubusercontent.com/100159466/203089454-25c4436a-31f1-40d3-af70-21d944aa7311.png)

Espere a sua conclusão e depois roda a outra parte no novo servidor secundário.

E pronto o Failover já está finalizado, rode o job de backup, copy e restore. Sua base deverá sair de Restoring para Stand by.
