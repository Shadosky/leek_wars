```php
  //-------------------//
 //------- INIT ------//
//-------------------//
var weapon = getWeapon();
if( weapon === null) {
	setWeapon(WEAPON_DOUBLE_GUN);
	weapon = getWeapon();
}
  //-------------------//
 //----DECLARATION----//
//-------------------//
var enemy = getNearestEnemy();
var enemyChip = getChips(enemy);
var enemyWeapon = getWeapon(enemy);
var enemyEffectWeapon = getWeaponEffects(enemyWeapon);
var enemyRange = getWeaponMaxRange(enemyWeapon);
var enemyHasSpark = (inArray(enemyChip, CHIP_SPARK));
var enemyLife = getLife(enemy);
var enemyDammageMultiplier = 1+(getStrength(enemy)/100);

var range = getWeaponMaxRange(weapon);
var spellRange = getChipMaxRange(CHIP_SPARK);
var myCell = getCell();
var hisCell = getCell(enemy);
var dontTest = false;

var hp = getLife();

var dammageMultiplier = 1+(getStrength()/100);
var effectWeapon = getWeaponEffects(weapon);
var potentialOutput = getLethal(weapon, effectWeapon, dammageMultiplier);
var enemyPotentialOutput = getLethal(enemyWeapon, enemyEffectWeapon, enemyDammageMultiplier);
var canLethal = (enemyLife < potentialOutput);

function getLethal(weapon, effectWeapon, dammageMultiplier) {
	if (weapon == WEAPON_PISTOL) {
		return (((effectWeapon[0][1]+effectWeapon[0][2])*dammageMultiplier)/2)*3;
	}
	if (weapon == WEAPON_DOUBLE_GUN) {
		var dmg1 = (((effectWeapon[0][1]+effectWeapon[0][2])*dammageMultiplier)/2)*2;
		var dmg2 = (((effectWeapon[1][1]+effectWeapon[1][2])*dammageMultiplier)/2)*2;
		return dmg1 + dmg2;
	}
}

function healer(cellToHeal) {
	var tp = getTP();
	if (canUseChipOnCell(CHIP_BANDAGE, cellToHeal)) {
		if (tp > 1) {
       		useChipOnCell(CHIP_BANDAGE, cellToHeal);
		}
		return true;
	}
	return false;
}

function findBlindSpot(movePoint, hisCell, myCell) {
	if (!lineOfSight(myCell, hisCell)) {
		return 0;
	}
	var myX = getCellX(myCell);
	var myY = getCellY(myCell);
	for(var i=0; i <= movePoint; i++) {
		// todo do it
	}
}

function testSolution(movePoint, hisCell, myCell) {
	var path = getPath(myCell, hisCell);
	if (lineOfSight(path[movePoint], hisCell)) {
		return movePoint;
	} else {
		// Todo check the other cells of the path
		for (var i = movePoint; i > 0 ; i--) {
			if (lineOfSight(path[i], hisCell)) {
				return i;
			}
		}
		return 0;
	}
}
function getSafeMove(myCell, hisCell, range, spellRange, enemyRange, enemyHasSpark, canLethal, @dontTest) {
	var mp = getTotalMP();
	var dist =  getCellDistance(myCell , hisCell);
	var mpToShoot = dist - range;
	var mpToCast = dist - spellRange;
	var enemyTrueRange = (enemyHasSpark && false)? 10 : enemyRange;
	var safeRange = dist - enemyTrueRange;
	
	if (mpToShoot <= 0) {
		return  0;
	}
	if(mpToShoot <= 3) {
		return mpToShoot;
	} else {
		if(mpToCast <= 3) {
			dontTest = true;
			return mpToCast;
		}
	}
	if (safeRange > mp) {
			dontTest = true;
			return safeRange-(mp+1);
		}
	debugE("ERROR");
}

function shootAt(enemy,canLethal) {
	var tp = getTP();
	if (canUseWeapon(enemy)) {
		if (canLethal) {
			say("I kill you !");
		}
		while (tp > 2) {
       		useWeapon(enemy);
        	tp = tp-3;
		}
		return true;
	}
	return false;
}

function burnIt(enemy) {
	var tp = getTP();
	if (canUseChip(CHIP_SPARK, enemy)) {
		while (tp > 2) {
       		useChip(CHIP_SPARK, enemy);
        	tp = tp-3;
		}
		return true;
	}
	return false;
}

  //------------------------//
 //---Routine start here---//
//------------------------//
debug(canLethal);
if(hp <= enemyPotentialOutput && (!canLethal || weapon == WEAPON_DOUBLE_GUN)){
	//don't miss a lethal because ur healing urself..
	// have to learned it the hard way
	// RIP MyAwesomePoiro 2016-2016
	healer(myCell);
}

var movePoint = getSafeMove(myCell, hisCell, range, spellRange, enemyRange, enemyHasSpark, canLethal, dontTest);
debug(movePoint);
if (movePoint > 0) {
	var testedMp = testSolution(movePoint, hisCell, myCell);
	if (testedMp != 0 || dontTest ){
		moveToward(enemy, movePoint);
		myCell = getCell();
		if (!shootAt(enemy, canLethal)) {
			burnIt(enemy);
		}
		healer(myCell);
		moveAwayFrom(enemy);
	} else {
		burnIt(enemy);
		healer(myCell);
		moveAwayFrom(enemy, movePoint);
	}
} else {
	if(!shootAt(enemy, canLethal)) {
		burnIt(enemy);
	}
	healer(myCell);
	moveAwayFrom(enemy);
}
```
