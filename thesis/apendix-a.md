## Додаток А - SQL-скрипт

```
SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='TRADITIONAL,ALLOW_INVALID_DATES';

-- -----------------------------------------------------
-- Schema utilities
-- -----------------------------------------------------

-- -----------------------------------------------------
-- Schema utilities
-- -----------------------------------------------------
CREATE SCHEMA IF NOT EXISTS `utilities` DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ;
USE `utilities` ;

-- -----------------------------------------------------
-- Table `utilities`.`user`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `utilities`.`user` (
  `idUser` INT UNSIGNED NOT NULL,
  `firstName` VARCHAR(30) NOT NULL,
  `middleName` VARCHAR(30) NULL,
  `lastName` VARCHAR(30) NOT NULL,
  `email` VARCHAR(100) NOT NULL,
  `password` VARCHAR(255) NOT NULL,
  `street` VARCHAR(25) NULL,
  `house` SMALLINT(3) NULL,
  `apartment` SMALLINT(3) NULL,
  `city` VARCHAR(25) NULL,
  `telephone` INT(10) NULL,
  `contract` DATE NULL,
  `dispatcher` TINYINT(1) NOT NULL,
  PRIMARY KEY (`idUser`),
  INDEX `user_idx` USING BTREE (`lastName` ASC))
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `utilities`.`profile`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `utilities`.`profile` (
  `idProfile` INT UNSIGNED NOT NULL,
  `name` VARCHAR(100) NOT NULL,
  `services` MEDIUMTEXT NULL,
  PRIMARY KEY (`idProfile`))
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `utilities`.`serviceTeam`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `utilities`.`serviceTeam` (
  `idServiceTeam` INT UNSIGNED NOT NULL,
  `name` VARCHAR(100) NOT NULL,
  `profile` INT NOT NULL,
  PRIMARY KEY (`idServiceTeam`),
  INDEX `fk_serviceTeam_profile1_idx` (`profile` ASC),
  CONSTRAINT `fk_serviceTeam_profile1`
    FOREIGN KEY (`profile`)
    REFERENCES `utilities`.`profile` (`idProfile`)
    ON DELETE RESTRICT
    ON UPDATE CASCADE)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `utilities`.`request`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `utilities`.`request` (
  `idRequest` BIGINT(20) UNSIGNED NOT NULL,
  `dateTime` DATETIME NOT NULL,
  `user` INT UNSIGNED NOT NULL,
  `profile` INT UNSIGNED NULL,
  `description` MEDIUMTEXT NOT NULL,
  `serviceTeam` INT UNSIGNED NULL,
  PRIMARY KEY (`idRequest`),
  INDEX `fk_request_user_idx` (`user` ASC),
  INDEX `fk_request_profile1_idx` (`profile` ASC),
  INDEX `fk_request_serviceTeam1_idx` (`serviceTeam` ASC),
  INDEX `description_idx` USING BTREE (`description` ASC),
  CONSTRAINT `fk_request_user`
    FOREIGN KEY (`user`)
    REFERENCES `utilities`.`user` (`idUser`)
    ON DELETE RESTRICT
    ON UPDATE CASCADE,
  CONSTRAINT `fk_request_profile1`
    FOREIGN KEY (`profile`)
    REFERENCES `utilities`.`profile` (`idProfile`)
    ON DELETE RESTRICT
    ON UPDATE CASCADE,
  CONSTRAINT `fk_request_serviceTeam1`
    FOREIGN KEY (`serviceTeam`)
    REFERENCES `utilities`.`serviceTeam` (`idServiceTeam`)
    ON DELETE RESTRICT
    ON UPDATE CASCADE)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `utilities`.`status`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `utilities`.`status` (
  `idStatus` BIGINT(20) UNSIGNED NOT NULL,
  `request` BIGINT(20) UNSIGNED NOT NULL,
  `status` TINYINT(1) NOT NULL,
  `dateTime` DATETIME NOT NULL,
  `comment` VARCHAR(255) NULL,
  PRIMARY KEY (`idStatus`),
  INDEX `fk_status_request1_idx` (`request` ASC),
  CONSTRAINT `fk_status_request1`
    FOREIGN KEY (`request`)
    REFERENCES `utilities`.`request` (`idRequest`)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `utilities`.`message`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `utilities`.`message` (
  `idMessage` INT UNSIGNED NOT NULL,
  `user` INT UNSIGNED NOT NULL,
  `request` BIGINT(20) UNSIGNED NOT NULL,
  `dateTime` DATETIME NOT NULL,
  `content` MEDIUMTEXT NOT NULL,
  PRIMARY KEY (`idMessage`),
  INDEX `fk_message_user1_idx` (`user` ASC),
  INDEX `fk_message_request1_idx` (`request` ASC),
  CONSTRAINT `fk_message_user1`
    FOREIGN KEY (`user`)
    REFERENCES `utilities`.`user` (`idUser`)
    ON DELETE CASCADE
    ON UPDATE CASCADE,
  CONSTRAINT `fk_message_request1`
    FOREIGN KEY (`request`)
    REFERENCES `utilities`.`request` (`idRequest`)
    ON DELETE CASCADE
    ON UPDATE CASCADE)
ENGINE = InnoDB;

USE `utilities`;

DELIMITER $$
USE `utilities`$$
CREATE TRIGGER `utilities`.`request_AFTER_INSERT`
	AFTER INSERT ON `request`
    FOR EACH ROW
BEGIN
	INSERT INTO `status` SET request = NEW.id, status = 1,
    dateTime = NOW();
END;$$

USE `utilities`$$
CREATE TRIGGER `utilities`.`request_AFTER_UPDATE`
	AFTER UPDATE ON `request`
    FOR EACH ROW
BEGIN
	IF NEW.serviceTeam <> NULL THEN  
		INSERT INTO `status` SET request = NEW.id, status = 3,
		dateTime = NOW();
	END IF;
END;$$


DELIMITER ;

SET SQL_MODE=@OLD_SQL_MODE;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;
```
