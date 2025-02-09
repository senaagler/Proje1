# Proje1
Savaş Oyunu ve Senaryolar
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>  // senaryoları kodumuza çekebilmemiz için gerekli cURL kütüphanesi

#define MAX_UNITS 100
#define MAX_NAME_LENGTH 50
#define MAX_HEROES 50
#define MAX_CREATURES 50




// Birim yapısı
typedef struct {
    char name[MAX_NAME_LENGTH];
    int attack;
    int defense;
    int health;
    int criticalChance;
    int count;
} Unit;

// Kahraman yapısı
typedef struct {
    char name[MAX_NAME_LENGTH];
    char bonusType[MAX_NAME_LENGTH];
    int bonusValue;
    char description[256];
} Hero;

// Canavar yapısı
   typedef struct {
    char name[MAX_NAME_LENGTH];
    char effectType[MAX_NAME_LENGTH];
    int effectValue;
    char description[256];
} Creature;


typedef struct {
    char name[MAX_NAME_LENGTH];
    int value;
    char description[256];
} Research;

typedef struct {
    Unit humanEmpire[MAX_UNITS];
    int humanCount;
    Unit orcLegion[MAX_UNITS];
    int orcCount;
    Hero heroes[MAX_HEROES];
    int heroCount;
    Creature creatures[MAX_CREATURES];
    int creatureCount;
    Research researches[20];
    int researchCount;
} GameData;



// Kahramanları yükleme fonksiyonu
void loadHeroes(GameData *gameData, const char *filename) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        printf("Dosya acilamadi: %s\n", filename);
        return;
    }

    char name[MAX_NAME_LENGTH];
    char bonusType[MAX_NAME_LENGTH];
    int bonusValue;
    char description[256];

    while (fscanf(file, "%s %s %d %[^\n]", name, bonusType, &bonusValue, description) != EOF) {
        Hero hero;
        strcpy(hero.name, name);
        strcpy(hero.bonusType, bonusType);
        hero.bonusValue = bonusValue;
        strcpy(hero.description, description);

        gameData->heroes[gameData->heroCount++] = hero;
    }

    fclose(file);
}
char *loadJsonFile(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        printf("Dosya acilamadi: %s\n", filename);
        return NULL;
    }

    fseek(file, 0, SEEK_END);
    long fileSize = ftell(file);
    rewind(file);

    char *jsonString = (char *)malloc(fileSize + 1);
    fread(jsonString, 1, fileSize, file);
    jsonString[fileSize] = '\0';

    fclose(file);
    return jsonString;
}
// Canavarları yükleme fonksiyonu
void loadCreatures(GameData *gameData, const char *filename) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        printf("Dosya acilamadi: %s\n", filename);
        return;
    }

    char name[MAX_NAME_LENGTH];
    char effectType[MAX_NAME_LENGTH];
    int effectValue;
    char description[256];

    while (fscanf(file, "%s %s %d %[^\n]", name, effectType, &effectValue, description) != EOF) {
        Creature creature;
        strcpy(creature.name, name);
        strcpy(creature.effectType, effectType);
        creature.effectValue = effectValue;
        strcpy(creature.description, description);

        gameData->creatures[gameData->creatureCount++] = creature;
    }

    fclose(file);
}
void loadResearch(GameData *gameData, const char *filename) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        printf("Dosya açilamadi: %s\n", filename);
        return;
    }

    char name[MAX_NAME_LENGTH];
    int value;
    char description[256];

    while (fscanf(file, "%s %d %[^\n]", name, &value, description) != EOF) {
        Research research;
        strcpy(research.name, name);
        research.value = value;
        strcpy(research.description, description);

        gameData->researches[gameData->researchCount++] = research;
    }

    fclose(file);
}

void loadScenario(GameData *gameData, const char *filename) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        printf("Senaryo dosyasi açilamadi: %s\n", filename);
        return;
    }

    char line[256];
    int isHuman = -1;

    while (fgets(line, sizeof(line), file)) {
        if (strstr(line, "InsanBasla") != NULL) {
            isHuman = 1;
            continue;
        } else if (strstr(line, "OrkBasla") != NULL) {
            isHuman = 0;
            continue;
        }

        if (isHuman == 1 || isHuman == 0) {
            char unitName[MAX_NAME_LENGTH];
            int attack, defense, health, criticalChance, count;

            if (sscanf(line, "%s %d %d %d %d %d", unitName, &attack, &defense, &health, &criticalChance, &count) == 6) {
                Unit unit;
                strcpy(unit.name, unitName);
                unit.attack = attack;
                unit.defense = defense;
                unit.health = health;
                unit.criticalChance = criticalChance;
                unit.count = count;

                if (isHuman == 1) {
                    gameData->humanEmpire[gameData->humanCount++] = unit;
                } else {
                    gameData->orcLegion[gameData->orcCount++] = unit;
                }
            }
        }

        if (strstr(line, "ArastirmaBasla") != NULL) {
            char researchName[MAX_NAME_LENGTH];
            int value;
            char description[256];

            if (fgets(line, sizeof(line), file) && sscanf(line, "%s %d %[^\n]", researchName, &value, description) == 3) {
                Research research;
                strcpy(research.name, researchName);
                research.value = value;
                strcpy(research.description, description);

                gameData->researches[gameData->researchCount++] = research;
            }
        }

        if (strstr(line, "KahramanBasla") != NULL) {
            char heroName[MAX_NAME_LENGTH];
            char bonusType[MAX_NAME_LENGTH];
            int bonusValue;
            char description[256];

            if (fgets(line, sizeof(line), file) && sscanf(line, "%s %s %d %[^\n]", heroName, bonusType, &bonusValue, description) == 4) {
                Hero hero;
                strcpy(hero.name, heroName);
                strcpy(hero.bonusType, bonusType);
                hero.bonusValue = bonusValue;
                strcpy(hero.description, description);

                gameData->heroes[gameData->heroCount++] = hero;
            }
        }

        if (strstr(line, "CanavarBasla") != NULL) {
            char creatureName[MAX_NAME_LENGTH];
            char effectType[MAX_NAME_LENGTH];
            int effectValue;
            char description[256];

            if (fgets(line, sizeof(line), file) && sscanf(line, "%s %s %d %[^\n]", creatureName, effectType, &effectValue, description) == 4) {
                Creature creature;
                strcpy(creature.name, creatureName);
                strcpy(creature.effectType, effectType);
                creature.effectValue = effectValue;
                strcpy(creature.description, description);

                gameData->creatures[gameData->creatureCount++] = creature;
            }
        }
    }

    fclose(file);
}
void parseHumanCreatures(const char *json, GameData *gameData) {
    char name[MAX_NAME_LENGTH];
    char effectType[MAX_NAME_LENGTH];
    int effectValue;
    char description[256];

    char *humanStart = strstr(json, "\"insan_imparatorlugu\"");
    if (humanStart != NULL) {
        char *creatureStart = strchr(humanStart, '{');

        while ((creatureStart = strstr(creatureStart, "{")) != NULL) {
            sscanf(strstr(creatureStart, "\"etki_degeri\""), "\"etki_degeri\": \"%d\"", &effectValue);
            sscanf(strstr(creatureStart, "\"etki_turu\""), "\"etki_turu\": \"%[^\"]\"", effectType);
            sscanf(strstr(creatureStart, "\"aciklama\""), "\"aciklama\": \"%[^\"]\"", description);

            char *nameStart = creatureStart - 1;
            while (*nameStart != '\"') nameStart--;
            nameStart++;
            sscanf(nameStart, "%[^\"]", name);

            Creature creature;
            strcpy(creature.name, name);
            strcpy(creature.effectType, effectType);
            creature.effectValue = effectValue;
            strcpy(creature.description, description);

            gameData->creatures[gameData->creatureCount++] = creature;

            creatureStart = strchr(creatureStart + 1, '{');
        }
    }
}
void parseResearchFromJson(const char *json, GameData *gameData) {
    char researchName[MAX_NAME_LENGTH];
    char level[MAX_NAME_LENGTH];
    int value;
    char description[256];

    char *researchStart = strstr(json, "{");
    while (researchStart != NULL) {
        sscanf(researchStart, " \"%[^\"]\"", researchName);

        char *levelStart = strstr(researchStart, "{");
        while ((levelStart = strstr(levelStart, "\"seviye_")) != NULL) {
            sscanf(levelStart, "\"seviye_%*[^\"]\"");


            sscanf(strstr(levelStart, "\"deger\""), "\"deger\": \"%d\"", &value);
            sscanf(strstr(levelStart, "\"aciklama\""), "\"aciklama\": \"%[^\"]\"", description);


            Research research;
            snprintf(research.name, sizeof(research.name), "%s_%s", researchName, level);
            research.value = value;
            strcpy(research.description, description);



            gameData->researches[gameData->researchCount++] = research;



            levelStart = strstr(levelStart + 1, "\"seviye_");
        }



        researchStart = strstr(researchStart + 1, "\"");
    }
}
void parseUnitsFromJson(const char *json, GameData *gameData) {
    char unitName[MAX_NAME_LENGTH];
    int attack, defense, health, criticalChance;



    char *humanStart = strstr(json, "\"insan_imparatorlugu\"");
    if (humanStart != NULL) {
        char *unitStart = strchr(humanStart, '{');

        while ((unitStart = strstr(unitStart, "{")) != NULL) {  // Her insan birimi için teker teker kullanıcaz
            sscanf(strstr(unitStart, "\"saldiri\""), "\"saldiri\": %d", &attack);
            sscanf(strstr(unitStart, "\"savunma\""), "\"savunma\": %d", &defense);
            sscanf(strstr(unitStart, "\"saglik\""), "\"saglik\": %d", &health);
            sscanf(strstr(unitStart, "\"kritik_sans\""), "\"kritik_sans\": %d", &criticalChance);



            char *nameStart = unitStart - 1;
            while (*nameStart != '\"') nameStart--;
            nameStart++;
            sscanf(nameStart, "%[^\"]", unitName);


            Unit unit;
            strcpy(unit.name, unitName);
            unit.attack = attack;
            unit.defense = defense;
            unit.health = health;
            unit.criticalChance = criticalChance;


            gameData->humanEmpire[gameData->humanCount++] = unit;

            // Sonraki birime geçen komut
            unitStart = strchr(unitStart + 1, '{');
        }
    }

    // Şimdi ork leginin birimlerini işleyelim
    char *orcStart = strstr(json, "\"ork_legi\"");
    if (orcStart != NULL) {
        char *unitStart = strchr(orcStart, '{');

        while ((unitStart = strstr(unitStart, "{")) != NULL) {  // Her ork birimi için teker teker kullanıcaz
            sscanf(strstr(unitStart, "\"saldiri\""), "\"saldiri\": %d", &attack);
            sscanf(strstr(unitStart, "\"savunma\""), "\"savunma\": %d", &defense);
            sscanf(strstr(unitStart, "\"saglik\""), "\"saglik\": %d", &health);
            sscanf(strstr(unitStart, "\"kritik_sans\""), "\"kritik_sans\": %d", &criticalChance);

            // Birim adını bulalım
            char *nameStart = unitStart - 1;
            while (*nameStart != '\"') nameStart--;
            nameStart++;
            sscanf(nameStart, "%[^\"]", unitName);

            // Yeni ork birimi oluşturalım
            Unit unit;
            strcpy(unit.name, unitName);
            unit.attack = attack;
            unit.defense = defense;
            unit.health = health;
            unit.criticalChance = criticalChance;

            // Birimi ork legine ekleyen kısım
            gameData->orcLegion[gameData->orcCount++] = unit;

            // Sonraki birime geçen komut
            unitStart = strchr(unitStart + 1, '{');
        }
    }
}

void parseOrcCreatures(const char *json, GameData *gameData) {
    char name[MAX_NAME_LENGTH];
    char effectType[MAX_NAME_LENGTH];
    int effectValue;
    char description[256];

    char *orcStart = strstr(json, "\"ork_legi\"");
    if (orcStart != NULL) {
        char *creatureStart = strchr(orcStart, '{');  // Orklar için açılış '{' bu şekilde yapılacak

        while ((creatureStart = strstr(creatureStart, "{")) != NULL) {  // Her yaratığı '{' ile bulacağız
            sscanf(strstr(creatureStart, "\"etki_degeri\""), "\"etki_degeri\": \"%d\"", &effectValue);
            sscanf(strstr(creatureStart, "\"etki_turu\""), "\"etki_turu\": \"%[^\"]\"", effectType);
            sscanf(strstr(creatureStart, "\"aciklama\""), "\"aciklama\": \"%[^\"]\"", description);

            // Yaratığın ismini bul (isim, anahtar kelime)
            char *nameStart = creatureStart - 1;
            while (*nameStart != '\"') nameStart--;  // İsimden önceki " karakterini bulacak
            nameStart++;
            sscanf(nameStart, "%[^\"]", name);

            // Yeni yaratık oluşturalım
            Creature creature;
            strcpy(creature.name, name);
            strcpy(creature.effectType, effectType);
            creature.effectValue = effectValue;
            strcpy(creature.description, description);

            // Yaratığı ork lejyonuna ekleyen komut
            gameData->creatures[gameData->creatureCount++] = creature;

            // Sonraki yaratığa geçen komut
            creatureStart = strchr(creatureStart + 1, '{');
        }
    }
}
void parseHeroesFromJson(const char *json, GameData *gameData) {
    char heroName[MAX_NAME_LENGTH];
    char bonusType[MAX_NAME_LENGTH];
    int bonusValue;
    char description[256];

    // İlk olarak insan imparatorluğu kahramanlarını işleyelim
    char *humanStart = strstr(json, "\"insan_imparatorlugu\"");
    if (humanStart != NULL) {
        char *heroStart = strchr(humanStart, '{');

        while ((heroStart = strstr(heroStart, "{")) != NULL) {  // Her insan kahramanı için tek tek kullanıcaz
            sscanf(strstr(heroStart, "\"bonus_degeri\""), "\"bonus_degeri\": \"%d\"", &bonusValue);
            sscanf(strstr(heroStart, "\"bonus_turu\""), "\"bonus_turu\": \"%[^\"]\"", bonusType);
            sscanf(strstr(heroStart, "\"aciklama\""), "\"aciklama\": \"%[^\"]\"", description);

            // Kahramanın adını bul (isim, anahtar kelime)
            char *nameStart = heroStart - 1;
            while (*nameStart != '\"') nameStart--;
            nameStart++;
            sscanf(nameStart, "%[^\"]", heroName);

            // Yeni kahraman oluşturalım
            Hero hero;
            strcpy(hero.name, heroName);
            strcpy(hero.bonusType, bonusType);
            hero.bonusValue = bonusValue;
            strcpy(hero.description, description);

            // Kahramanı gameData'ya ekleyen kısım
            gameData->heroes[gameData->heroCount++] = hero;

            // Sonraki kahramana geçen komut
            heroStart = strchr(heroStart + 1, '{');
        }
    }

    // Şimdi ork leginin kahramanlarını işleyelim
    char *orcStart = strstr(json, "\"ork_legi\"");
    if (orcStart != NULL) {
        char *heroStart = strchr(orcStart, '{');

        while ((heroStart = strstr(heroStart, "{")) != NULL) {  // Her ork kahramanı için...
            sscanf(strstr(heroStart, "\"bonus_degeri\""), "\"bonus_degeri\": \"%d\"", &bonusValue);
            sscanf(strstr(heroStart, "\"bonus_turu\""), "\"bonus_turu\": \"%[^\"]\"", bonusType);
            sscanf(strstr(heroStart, "\"aciklama\""), "\"aciklama\": \"%[^\"]\"", description);

            // Kahramanın adını bulan kısım
            char *nameStart = heroStart - 1;
            while (*nameStart != '\"') nameStart--;
            nameStart++;
            sscanf(nameStart, "%[^\"]", heroName);

            // Yeni kahraman oluşturan kısım
            Hero hero;
            strcpy(hero.name, heroName);
            strcpy(hero.bonusType, bonusType);
            hero.bonusValue = bonusValue;
            strcpy(hero.description, description);

            // Kahramanı gameData'ya ekleyen komut
            gameData->heroes[gameData->heroCount++] = hero;

            // Sonraki kahramana geçen komut
            heroStart = strchr(heroStart + 1, '{');
        }
    }
}
void initializeHumanEmpire(GameData *gameData) {
    strcpy(gameData->humanEmpire[0].name, "Piyadeler");
    gameData->humanEmpire[0].attack = 30;
    gameData->humanEmpire[0].defense = 60;
    gameData->humanEmpire[0].health = 120;
    gameData->humanEmpire[0].criticalChance = 5;
    gameData->humanEmpire[0].count = 500;

    strcpy(gameData->humanEmpire[1].name, "Okçular");
    gameData->humanEmpire[1].attack = 40;
    gameData->humanEmpire[1].defense = 30;
    gameData->humanEmpire[1].health = 90;
    gameData->humanEmpire[1].criticalChance = 10;
    gameData->humanEmpire[1].count = 300;

    gameData->humanCount = 2;  // İnsan birimi sayısı burada doğru ayarlanmalı.
}

void initializeOrcLegion(GameData *gameData) {
    // Ork Dövüşçüleri
    strcpy(gameData->orcLegion[0].name, "Ork Dovusculeri");
    gameData->orcLegion[0].attack = 25;
    gameData->orcLegion[0].defense = 30;  // Savunma artırıldı
    gameData->orcLegion[0].health = 120;  // Sağlık artırıldı
    gameData->orcLegion[0].criticalChance = 8;
    gameData->orcLegion[0].count = 600;

    // Mızrakçılar
    strcpy(gameData->orcLegion[1].name, "Mizrakcilar");
    gameData->orcLegion[1].attack = 30;
    gameData->orcLegion[1].defense = 35;  // Savunma artırıldı
    gameData->orcLegion[1].health = 110;  // Sağlık artırıldı
    gameData->orcLegion[1].criticalChance = 5;
    gameData->orcLegion[1].count = 400;

    // Varg Binicileri
    strcpy(gameData->orcLegion[2].name, "Varg Binicileri");
    gameData->orcLegion[2].attack = 40;
    gameData->orcLegion[2].defense = 40;  // Savunma artırıldı
    gameData->orcLegion[2].health = 150;  // Sağlık artırıldı
    gameData->orcLegion[2].criticalChance = 6;
    gameData->orcLegion[2].count = 200;

    // Troller
    strcpy(gameData->orcLegion[3].name, "Troller");
    gameData->orcLegion[3].attack = 70;
    gameData->orcLegion[3].defense = 50;  // Savunma artırıldı
    gameData->orcLegion[3].health = 220;  // Sağlık artırıldı
    gameData->orcLegion[3].criticalChance = 5;
    gameData->orcLegion[3].count = 100;

    gameData->orcCount = 4;
}


int calculateTotalAttack(GameData *gameData, int isHuman) {
    int totalAttack = 0;

    // İnsan birimlerini hesaplayan kısım
    if (isHuman) {
        for (int i = 0; i < gameData->humanCount; i++) {
            int unitAttack = gameData->humanEmpire[i].attack; // Birim saldırı gücü
            totalAttack += unitAttack; // Toplam saldırı gücüne ekleyen denklem
        }
    }
    // Ork birimlerini hesaplayan kısım
    else {
        for (int i = 0; i < gameData->orcCount; i++) {
            int unitAttack = gameData->orcLegion[i].attack; // Birim saldırı gücü
            totalAttack += unitAttack; // Toplam saldırı gücüne ekle
        }
    }

    // Kahraman bonuslarını ekleyen kısım
    for (int i = 0; i < gameData->heroCount; i++) {
        if (strcmp(gameData->heroes[i].bonusType, "attack") == 0) {
            totalAttack += gameData->heroes[i].bonusValue; // Kahraman saldırı bonusu
        }
    }

    // Araştırma bonuslarını ekle kısım
    for (int i = 0; i < gameData->researchCount; i++) {
        if (strstr(gameData->researches[i].name, "saldiri_gelistirmesi") != NULL) {
            totalAttack += totalAttack * gameData->researches[i].value / 100; // Araştırma saldırı bonusu denklemi
        }
    }

    return totalAttack;
}

int calculateTotalDefense(GameData *gameData, int isHuman) {
    int totalDefense = 0;

    if (isHuman) {
        for (int i = 0; i < gameData->humanCount; i++) {
            int unitDefense = gameData->humanEmpire[i].defense * gameData->humanEmpire[i].count;  // Savunma gücü * birim sayısı
            totalDefense += unitDefense;
        }
    } else {
        for (int i = 0; i < gameData->orcCount; i++) {
            int unitDefense = gameData->orcLegion[i].defense * gameData->orcLegion[i].count;
            totalDefense += unitDefense;
        }
    }

    // Kahraman bonuslarını ekleyen kısım
    for (int i = 0; i < gameData->heroCount; i++) {
        if (strcmp(gameData->heroes[i].bonusType, "defense") == 0) {
            totalDefense += gameData->heroes[i].bonusValue;
        }
    }

    // Araştırma bonuslarını ekleyen kısım
    for (int i = 0; i < gameData->researchCount; i++) {
        if (strstr(gameData->researches[i].name, "savunma_ustaligi") != NULL) {
            totalDefense += totalDefense * gameData->researches[i].value / 100;
        }
    }

    return totalDefense;
}
int calculateCriticalDamage(int baseDamage, int criticalChance) {
    int randomValue = rand() % 100;
    if (randomValue < criticalChance) {
        return baseDamage * 1.5;  // Kritik hasar %50 artırılır
    }
    return baseDamage;
}

int calculateTotalAttackWithCritical(GameData *gameData, int isHuman) {
    int totalAttack = 0;

    // İnsan veya ork birimlerini hesaplayan yer
    if (isHuman) {
        for (int i = 0; i < gameData->humanCount; i++) {
            int unitAttack = gameData->humanEmpire[i].attack * gameData->humanEmpire[i].count;  // Saldırı gücü * birim sayısı
            totalAttack += unitAttack;
        }
    } else {
        for (int i = 0; i < gameData->orcCount; i++) {
            int unitAttack = gameData->orcLegion[i].attack * gameData->orcLegion[i].count;
            totalAttack += unitAttack;
        }
    }

    // Kahraman bonuslarını ekleyen yer
    for (int i = 0; i < gameData->heroCount; i++) {
        if (strcmp(gameData->heroes[i].bonusType, "attack") == 0) {
            totalAttack += gameData->heroes[i].bonusValue;
        }
    }

    // Araştırma bonuslarını ekleyen yer
    for (int i = 0; i < gameData->researchCount; i++) {
        if (strstr(gameData->researches[i].name, "saldiri_gelistirmesi") != NULL) {
            totalAttack += totalAttack * gameData->researches[i].value / 100; // Araştırma saldırı bonusu
        }
    }

    return totalAttack;
}
void distributeDamage(double netHasar, Unit *units, int *unitCount) {
    double toplamSavunmaGucu = 0;

    // Toplam savunma gücünü hesapla
    for (int i = 0; i < *unitCount; i++) {
        toplamSavunmaGucu += units[i].defense * units[i].count;
    }

    // Her bir birime hasar dağıtımı yap
    for (int i = 0; i < *unitCount; i++) {
        if (units[i].count <= 0) continue;  // Eğer birim sayısı sıfırsa hasar uygulanmaz

        double oran = (units[i].defense * units[i].count) / toplamSavunmaGucu;
        double hasarDagilimi = netHasar * oran;

        int birimKaybi = (int)(hasarDagilimi / units[i].health);
        if (birimKaybi > units[i].count) {
            birimKaybi = units[i].count;  // Birim sayısı sıfırın altına düşmesin
        }

        units[i].count -= birimKaybi;

        printf("%s: %d birim kaybedildi. Kalan birim sayisi: %d\n", units[i].name, birimKaybi, units[i].count);

        if (units[i].count <= 0) {
            printf("%s birimi yok oldu!\n", units[i].name);
            units[i].count = 0;  // Birim tamamen yok olur
        }
    }
}

void updateUnitHealth(GameData *gameData, int isHuman, int totalDamage) {
    if (totalDamage <= 0) {
        return;  // Negatif veya sıfır hasar uygulanmıyor
    }

    Unit *units = isHuman ? gameData->humanEmpire : gameData->orcLegion;
    int *unitCount = isHuman ? &gameData->humanCount : &gameData->orcCount;

    for (int i = 0; i < *unitCount; i++) {
        if (units[i].count <= 0) continue;  // Eğer birim kalmamışsa bu birime hasar uygulanmaz

        int totalUnitHealth = units[i].health * units[i].count;
        int damagePerUnit = totalDamage / units[i].count;

        if (damagePerUnit > units[i].health) {
            damagePerUnit = units[i].health;  // Maksimum hasar birim sağlığını aşmasın
        }

        units[i].health -= damagePerUnit;

        printf("%s: Birim Basi Saglik: %d, Uygulanan Hasar: %d\n", units[i].name, units[i].health, damagePerUnit);

        if (units[i].health <= 0) {
            printf("%s birimi yok oldu!\n", units[i].name);
            units[i].count = 0;  // Birim tamamen yok olur
        }
    }
}
void applyFatigue(GameData *gameData, int isHuman, int turn) {
    if (turn % 5 == 0) {
        if (isHuman) {
            for (int i = 0; i < gameData->humanCount; i++) {
                gameData->humanEmpire[i].attack *= 0.90;  // %10 azalma
                gameData->humanEmpire[i].defense *= 0.90; // %10 azalma
            }
        } else {
            for (int i = 0; i < gameData->orcCount; i++) {
                gameData->orcLegion[i].attack *= 0.90;
                gameData->orcLegion[i].defense *= 0.90;
            }
        }
    }
}

void applyHeroAndCreatureBonuses(GameData *gameData, int isHuman) {
    if (isHuman) {
        // Kahraman bonusları
        for (int i = 0; i < gameData->heroCount; i++) {
            Hero *hero = &gameData->heroes[i];
            float bonusMultiplier = 1.0f + (hero->bonusValue / 100.0f);

            if (strcmp(hero->bonusType, "defense") == 0) {
                for (int j = 0; j < gameData->humanCount; j++) {
                    gameData->humanEmpire[j].defense *= bonusMultiplier;
                }
            } else if (strcmp(hero->bonusType, "attack") == 0) {
                for (int j = 0; j < gameData->humanCount; j++) {
                    gameData->humanEmpire[j].attack *= bonusMultiplier;
                }
            }
        }

        // Canavar bonusları
        for (int i = 0; i < gameData->creatureCount; i++) {
            Creature *creature = &gameData->creatures[i];
            float bonusMultiplier = 1.0f + (creature->effectValue / 100.0f);

            if (strcmp(creature->effectType, "defense") == 0) {
                for (int j = 0; j < gameData->humanCount; j++) {
                    gameData->humanEmpire[j].defense *= bonusMultiplier;
                }
            } else if (strcmp(creature->effectType, "attack") == 0) {
                for (int j = 0; j < gameData->humanCount; j++) {
                    gameData->humanEmpire[j].attack *= bonusMultiplier;
                }
            }
        }
    } else {
        // Aynı işlemler orklar için de yapılır
        for (int i = 0; i < gameData->heroCount; i++) {
            Hero *hero = &gameData->heroes[i];
            float bonusMultiplier = 1.0f + (hero->bonusValue / 100.0f);

            if (strcmp(hero->bonusType, "defense") == 0) {
                for (int j = 0; j < gameData->orcCount; j++) {
                    gameData->orcLegion[j].defense *= bonusMultiplier;
                }
            } else if (strcmp(hero->bonusType, "attack") == 0) {
                for (int j = 0; j < gameData->orcCount; j++) {
                    gameData->orcLegion[j].attack *= bonusMultiplier;
                }
            }
        }

        // Canavar bonusları orklara da uygulanır
        for (int i = 0; i < gameData->creatureCount; i++) {
            Creature *creature = &gameData->creatures[i];
            float bonusMultiplier = 1.0f + (creature->effectValue / 100.0f);

            if (strcmp(creature->effectType, "defense") == 0) {
                for (int j = 0; j < gameData->orcCount; j++) {
                    gameData->orcLegion[j].defense *= bonusMultiplier;
                }
            } else if (strcmp(creature->effectType, "attack") == 0) {
                for (int j = 0; j < gameData->orcCount; j++) {
                    gameData->orcLegion[j].attack *= bonusMultiplier;
                }
            }
        }
    }
}

void applyResearchBonuses(GameData *gameData) {
    for (int i = 0; i < gameData->researchCount; i++) {
        Research *research = &gameData->researches[i];

        if (strstr(research->name, "savunma_ustaligi") != NULL) {

            for (int j = 0; j < gameData->humanCount; j++) {
                gameData->humanEmpire[j].defense += gameData->humanEmpire[j].defense * research->value / 100;
            }
            for (int j = 0; j < gameData->orcCount; j++) {
                gameData->orcLegion[j].defense += gameData->orcLegion[j].defense * research->value / 100;
            }
        }

        if (strstr(research->name, "saldiri_gelistirmesi") != NULL) {

            for (int j = 0; j < gameData->humanCount; j++) {
                gameData->humanEmpire[j].attack += gameData->humanEmpire[j].attack * research->value / 100;
            }
            for (int j = 0; j < gameData->orcCount; j++) {
                gameData->orcLegion[j].attack += gameData->orcLegion[j].attack * research->value / 100;
            }
        }
    }
}
int calculateNetDamage(int totalAttack, int totalDefense) {
    if (totalAttack == 0) {
        return 0;
    }

    double reductionFactor = (double)totalDefense / (double)totalAttack;

    if (reductionFactor > 1.0) {
        reductionFactor = 0.9;
    }

    double netDamage = totalAttack * (1.0 - reductionFactor);
    return (int)netDamage;
}

void printGameData(const GameData *gameData) {
    printf("Insan Imparatorlugu Birimleri:\n");
    for (int i = 0; i < gameData->humanCount; i++) {
        if (gameData->humanEmpire[i].count > 0) {
            printf("Birim: %s, Kalan: %d\n", gameData->humanEmpire[i].name, gameData->humanEmpire[i].count);
        }
    }

    printf("Ork Legi Birimleri:\n");
    for (int i = 0; i < gameData->orcCount; i++) {
        if (gameData->orcLegion[i].count > 0) {
            printf("Birim: %s, Kalan: %d\n", gameData->orcLegion[i].name, gameData->orcLegion[i].count);
        }
    }
}

void battle(GameData *gameData) {
    int turn = 1;
    int attackingIsHuman = 1;
    FILE *battleLog = fopen("savas_sim.txt", "w");

    if (battleLog == NULL) {
        printf("Dosya acilamadi!\n");
        return;
    }

    while (gameData->humanCount > 0 && gameData->orcCount > 0) {
        fprintf(battleLog, "Tur %d basladi.\n", turn);

        if (attackingIsHuman) {
            int humanTotalAttack = calculateTotalAttackWithCritical(gameData, 1);
            int orcTotalDefense = calculateTotalDefense(gameData, 0);
            int netHasar = calculateNetDamage(humanTotalAttack, orcTotalDefense);

            printf("Tur %d: Insanlarin Toplam Saldirisi: %d, Orklarin Toplam Savunmasi: %d, Net Hasar: %d\n",
                   turn, humanTotalAttack, orcTotalDefense, netHasar);

            updateUnitHealth(gameData, 0, netHasar);
            distributeDamage(netHasar, gameData->orcLegion, &gameData->orcCount);

        } else {
            int orcTotalAttack = calculateTotalAttackWithCritical(gameData, 0);
            int humanTotalDefense = calculateTotalDefense(gameData, 1);
            int netHasarOrk = calculateNetDamage(orcTotalAttack, humanTotalDefense);

            printf("Tur %d: Orklarin Toplam Saldirisi: %d, Insanların Toplam Savunması: %d, Net Hasar: %d\n",
                   turn, orcTotalAttack, humanTotalDefense, netHasarOrk);

            updateUnitHealth(gameData, 1, netHasarOrk);
            distributeDamage(netHasarOrk, gameData->humanEmpire, &gameData->humanCount);
        }

        int humanRemaining = 0;
        int orcRemaining = 0;

        for (int i = 0; i < gameData->humanCount; i++) {
            if (gameData->humanEmpire[i].count > 0) {
                humanRemaining++;
            }
        }

        for (int i = 0; i < gameData->orcCount; i++) {
            if (gameData->orcLegion[i].count > 0) {
                orcRemaining++;
            }
        }

        if (humanRemaining == 0) {
            fprintf(battleLog, "Ork Lejyonu kazandi!\n");
            printf("Ork Lejyonu kazandi!\n");
            break;
        } else if (orcRemaining == 0) {
            fprintf(battleLog, "Insan Imparatorlugu kazandi!\n");
            printf("Insan Imparatorlugu kazandi!\n");
            break;
        }

        attackingIsHuman = !attackingIsHuman;
        turn++;
    }

    fclose(battleLog);
}

int main() {
    GameData gameData = {0};

    CURL *curl;
    CURLcode res;

    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();

    char *jsonData = loadJsonFile("creatures.json");
    if (jsonData != NULL) {
        parseHumanCreatures(jsonData, &gameData);
        parseOrcCreatures(jsonData, &gameData);
        parseResearchFromJson(jsonData, &gameData);
        parseUnitsFromJson(jsonData, &gameData);
        parseHeroesFromJson(jsonData, &gameData);
        printGameData(&gameData);
        free(jsonData);
    }

    if (curl) {
        int scenario_number;
        printf("1 ile 10 arasinda bir senaryo numarasi secin: ");
        scanf("%d", &scenario_number);

        char *url;
        if (scenario_number >= 1 && scenario_number <= 10) {
            url = (scenario_number == 1) ? "https://yapbenzet.org.tr/1.json" :
                  (scenario_number == 2) ? "https://yapbenzet.org.tr/2.json" :
                  (scenario_number == 3) ? "https://yapbenzet.org.tr/3.json" :
                  (scenario_number == 4) ? "https://yapbenzet.org.tr/4.json" :
                  (scenario_number == 5) ? "https://yapbenzet.org.tr/5.json" :
                  (scenario_number == 6) ? "https://yapbenzet.org.tr/6.json" :
                  (scenario_number == 7) ? "https://yapbenzet.org.tr/7.json" :
                  (scenario_number == 8) ? "https://yapbenzet.org.tr/8.json" :
                  (scenario_number == 9) ? "https://yapbenzet.org.tr/9.json" :
                  (scenario_number == 10)? "https://yapbenzet.org.tr/10.json": "";
        } else {
            printf("Geçersiz senaryo numarasi.\n");
            curl_easy_cleanup(curl);
            curl_global_cleanup();
            return 1;
        }

        curl_easy_setopt(curl, CURLOPT_URL, url);
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0L);
        curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0L);

        res = curl_easy_perform(curl);
        if (res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", curl_easy_strerror(res));
        }

        curl_easy_cleanup(curl);
    }

    curl_global_cleanup();

    initializeHumanEmpire(&gameData);
    initializeOrcLegion(&gameData);

    battle(&gameData);  // Tüm savaş işlemleri burada gerçekleştiriliyor
    printGameData(&gameData);

    return 0;
}
