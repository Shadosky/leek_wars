```php
  //-------------------//
 //------- INIT ------//
//-------------------//
var weapon = getWeapon();
if( weapon == null) {
        setWeapon(WEAPON_PISTOL);
        weapon = getWeapon();
}
  //-------------------//
 //----DECLARATION----//
//-------------------//
var enemy = getNearestEnemy();
var enemyChip = getChips(enemy);
var enemyWeapon = getWeapon(enemy);
var enemyRange = getWeaponMaxRange(enemyWeapon);
var enemyHasSpark = (inArray(enemyChip, CHIP_SPARK));
var enemyLife = getLife(enemy);

var range = getWeaponMaxRange(weapon);
var spellRange = getChipMaxRange(CHIP_SPARK);
var myCell = getCell();
var hisCell = getCell(enemy);

var dammageMultiplier = 1+(getStrength()/100);
var effectWeapon = getWeaponEffects(weapon);
var potentialOutput = (((effectWeapon[0][1]+effectWeapon[0][2])*dammageMultiplier)/2)*3;
var canLethal = (enemyLife < potentialOutput);

function testSolution(movePoint, hisCell, myCell) {
	var path = getPath(myCell, hisCell);
	if (lineOfSight(path[movePoint], hisCell)) {
		return true;
	} else {
		// Todo check the other cells of the path
		return false;
	}
}
function getSafeMove(myCell, hisCell, range, enemyRange, enemyHasSpark, canLethal) {
	var mp = getTotalMP();
	var dist =  getCellDistance(myCell , hisCell);
	var inRange = dist - range;
	var enemyTrueRange = (enemyHasSpark)? 10 : enemyRange;
	var safeRange = dist - enemyTrueRange;
	
	if (inRange <= 0) {
		return  0;
	}
	if (!canLethal && safeRange > mp) {
		if (safeRange < mp*2+1){
			return 9999;
		}
	}
	return inRange;
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

var usedRange = (inArray(enemyChip, CHIP_SPARK) || canLethal) ? range : spellRange;
var movePoint = getSafeMove(myCell, hisCell, usedRange, enemyRange, enemyHasSpark, canLethal);
if (movePoint == 9999) {
	var thisCell = getCell();
	moveToward(enemy, 1);
	shootAt(enemy, canLethal);
	burnIt(enemy);
	moveTowardCell(thisCell);
} else {
	if (movePoint > 0) {
		if (testSolution(movePoint, hisCell, myCell)){
			moveToward(enemy, movePoint);
			if (!shootAt(enemy, canLethal)) {
				burnIt(enemy);
				moveAwayFrom(enemy);
			}
		}
	} else {
		if(!shootAt(enemy, canLethal)) {
			burnIt(enemy);
		}
		moveAwayFrom(enemy);
	}
}
```
