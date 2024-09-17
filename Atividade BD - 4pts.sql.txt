
-- Criando as tabelas

CREATE TABLE Pacientes (
    id_paciente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    especie VARCHAR(50) NOT NULL,
    idade INT NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Veterinarios (
    id_veterinario INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    especialidade VARCHAR(50) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Consultas (
    id_consulta INT AUTO_INCREMENT PRIMARY KEY,
    id_paciente INT NOT NULL,
    id_veterinario INT NOT NULL,
    data_consulta DATE NOT NULL,
    custo DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_paciente) REFERENCES Pacientes(id_paciente),
    FOREIGN KEY (id_veterinario) REFERENCES Veterinarios(id_veterinario)
) ENGINE=InnoDB;

--  Criação de Stored Procedure

--   Procedure Agenda

DELIMITER $$

CREATE PROCEDURE agendar_consulta (
    IN p_id_paciente INT,
    IN p_id_veterinario INT,
    IN p_data_consulta DATE,
    IN p_custo DECIMAL(10, 2)
)
BEGIN
    INSERT INTO Consultas (id_paciente, id_veterinario, data_consulta, custo)
    VALUES (p_id_paciente, p_id_veterinario, p_data_consulta, p_custo);
END $$

DELIMITER ;

--  Procedure de Paciente

DELIMITER $$

CREATE PROCEDURE atualizar_paciente (
    IN p_id_paciente INT,
    IN p_novo_nome VARCHAR(100),
    IN p_nova_especie VARCHAR(50),
    IN p_nova_idade INT
)
BEGIN
    UPDATE Pacientes
    SET nome = p_novo_nome,
        especie = p_nova_especie,
        idade = p_nova_idade
    WHERE id_paciente = p_id_paciente;
END $$

DELIMITER ;

--  Procedure de Paciente

DELIMITER $$

CREATE PROCEDURE remover_consulta (
    IN p_id_consulta INT
)
BEGIN
    DELETE FROM Consultas
    WHERE id_consulta = p_id_consulta;
END $$

DELIMITER ;


-- Criação de Function

DELIMITER $$

CREATE FUNCTION total_gasto_paciente (
    p_id_paciente INT
) 
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total_gasto DECIMAL(10, 2);

    SELECT SUM(custo) INTO total_gasto
    FROM Consultas
    WHERE id_paciente = p_id_paciente;

    RETURN IFNULL(total_gasto, 0);
END $$

DELIMITER ;


-- Criação de Trigger 

-- Trigger Verificar identidade do Paciente

DELIMITER $$

CREATE TRIGGER verificar_idade_paciente
BEFORE INSERT ON Pacientes
FOR EACH ROW
BEGIN
   
    IF NEW.idade <= 0 THEN
       
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'O paciente deve ter a idade em numero positivo.';
    END IF;
END $$

DELIMITER ; 

-- Trigger Atualizar Custo de Consulta 


CREATE TABLE Log_Consultas (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    id_consulta INT NOT NULL,
    custo_antigo DECIMAL(10, 2) NOT NULL,
    custo_novo DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_consulta) REFERENCES Consultas(id_consulta)
) ENGINE=InnoDB;

DELIMITER $$

CREATE TRIGGER atualizar_custo_consulta
AFTER UPDATE ON Consultas
FOR EACH ROW
BEGIN

    IF OLD.custo <> NEW.custo THEN
   
tabela de log
        INSERT INTO Log_Consultas (id_consulta, custo_antigo, custo_novo)
        VALUES (NEW.id_consulta, OLD.custo, NEW.custo);
    END IF;
END $$

DELIMITER ;
