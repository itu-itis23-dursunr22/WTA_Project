# PSO ve GENETIC Algoritmaları Uygulama Planı

## 1. PSO (Particle Swarm Optimization) Mantığı
PSO'da "Arılar" yerine "Parçacıklar (Particles)" vardır.
1. **Konum (Position):** Sizin ABC'deki `population` matrisinizle tamamen aynıdır (Örn: `[1, 2, 1, 3...]`).
2. **Hız (Velocity):** PSO'ya özgü bir kavramdır. Parçacığın hedefler arasında nasıl yer değiştireceğini belirler.
3. **pBest (Personal Best):** Bir parçacığın kendi tarihi boyunca bulduğu en iyi konum.
4. **gBest (Global Best):** Tüm sürünün o ana kadar bulduğu en iyi konum.

---

## 2. GA (Genetik Algoritma) Mantığı ve WTA Uyarlaması
Genetik Algoritma (GA), evrimsel biyolojiden ilham alır. Bireyler (Kromozomlar) eşleşerek (Crossover) yeni nesiller (Çocuklar) üretir ve bazen mutasyon (Mutation) geçirirler.

1. **Popülasyon (Kromozomlar):** ABC ve PSO'daki "Konum" ile birebir aynıdır (1 ile `num_targets` arası tam sayılar). Her bir çözüm bir "Kromozom", her bir füze ataması ise bir "Gen"dir.
2. **Seçilim (Selection - Tournament):** Yeni nesli üretmek için anne ve baba seçilmelidir. Rastgele 2-3 birey seçilir, maliyeti (cost) en düşük olan "Ebeveyn" olarak kazanır.
3. **Çaprazlama (Crossover):** Anne ve babanın genleri karıştırılarak bir çocuk üretilir (Örn: 1., 3. ve 5. füzeyi anneden, 2. ve 4. füzeyi babadan almak gibi).
4. **Mutasyon (Mutation):** Çocuğun genlerinden (füzelerinden) bazıları çok düşük bir ihtimalle rastgele başka hedeflere kayar.
5. **Elitizm (Elitism):** En iyi birey kaybolmasın diye hiçbir değişikliğe uğramadan doğrudan bir sonraki nesle kopyalanır.

## User Review Required
Lütfen aşağıda hazırladığım `run_GENETIC.m` iskeletini inceleyin. İçinde hiçbir MATLAB kodu yoktur, sadece algoritmik talimatlar (yorum satırları) mevcuttur. Eğer mantık uygunsa "Proceed" butonuna basarak onaylayıp kodlamaya geçebilirsiniz.

---

### [NEW] `run_GENETIC.m` Dosyası İskeleti

```matlab
function [best_assignment, best_cost, convergence] = run_GENETIC(num_missiles, num_targets, P_matrix, target_threats, pop_size, max_iter, grouping_type, fixed_req, adaptive_req, penalty_factors)
% RUN_GENETIC Genetik Algoritma (GA) ile WTA problemini çözer.

arguments (Input)
    num_missiles    (1,1) double
    num_targets     (1,1) double
    P_matrix        (:,:) double
    target_threats  (1,:) double
    pop_size        (1,1) double % Popülasyon büyüklüğü (Çift sayı olması tercih edilir)
    max_iter        (1,1) double % Maksimum jenerasyon (döngü)
    grouping_type   (1,:) char
    fixed_req       (1,:) double
    adaptive_req    (:,:) double
    penalty_factors (1,4) double
end

arguments (Output)
    best_assignment (1,:) double % Bulunan en iyi kromozom (atama)
    best_cost       (1,1) double % En düşük maliyet
    convergence     (:,1) double % Her iterasyondaki en iyi değeri tutan dizi
end

% GA Özel Parametreleri
crossover_rate = 0.8; % Çaprazlama ihtimali
mutation_rate  = 0.1; % Mutasyon ihtimali
tournament_size = 3;  % Seçilim (Turnuva) büyüklüğü

% ---------------------------------------------------------
% 1. İLKLENDİRME (INITIALIZATION)
% ---------------------------------------------------------
% a) Başlangıç Popülasyonunu Üretin: 
%    1 ile num_targets arasında, pop_size x num_missiles boyutunda tam sayı matrisi (randi kullanabilirsiniz).

% b) Popülasyondaki Her Bireyin Maliyetini (Cost) Hesaplayın:
%    Bir for döngüsü ile fitness_function'ı çağırıp cost_array içine kaydedin.

% c) Başlangıçtaki En İyi Bireyi (best_assignment ve best_cost) Bulun.
%    min() fonksiyonu ile en düşük cost'u ve indeksini bulabilirsiniz.

% convergence dizisini sıfırlar ile başlatın.


% ---------------------------------------------------------
% 2. ANA EVRİM DÖNGÜSÜ (GENERATIONS)
% ---------------------------------------------------------
for iter = 1:max_iter
    
    % Yeni nesli tutmak için boş bir matris oluşturun (new_population).
    % Boyutu eski popülasyonla aynı (pop_size x num_missiles) olmalı.
    
    % --- ELİTİZM (En iyiyi koruma) ---
    % new_population'ın 1. satırına, şu ana kadarki en iyi çözümü (best_assignment) kopyalayın.
    % Böylece en iyi çözüm mutasyona uğrayıp bozulmaktan kurtulur.
    
    % --- YENİ NESLİ ÜRETME (SEÇİLİM, ÇAPRAZLAMA, MUTASYON) ---
    % Popülasyonun geri kalanını (2'den pop_size'a kadar) doldurmak için döngü açın:
    % (Not: Her adımda 1 çocuk üreteceğimiz için for i = 2 : pop_size şeklinde dönebilirsiniz)
    for i = 2:pop_size
        
        % 1. SEÇİLİM (Turnuva - Tournament Selection)
        % Anne için: Rastgele "tournament_size" (örn: 3) kadar birey seçin. 
        % İçlerinden cost değeri en küçük olanı "Anne (Parent 1)" yapın.
        % Baba için: Aynı işlemi tekrarlayıp "Baba (Parent 2)" yapın.
        
        % 2. ÇAPRAZLAMA (Crossover)
        % Eğer rand() < crossover_rate ise:
        % Anne ve Babanın genlerini karıştırarak 1 tane Çocuk (Child) oluşturun.
        % Ayrık problemlerde en iyi yöntem "Uniform Crossover"dır:
        % Çocuğun her bir füzesi (geni) için %50 ihtimalle Annenin o füzesini,
        % %50 ihtimalle Babanın o füzesini almasını sağlayın.
        % Eğer crossover olmazsa (ihtimal tutmazsa), çocuğu direkt Anne'ye eşitleyin.
        
        % 3. MUTASYON (Mutation)
        % Çocuğun içindeki füzeler için bir döngü açın (1'den num_missiles'a kadar).
        % Eğer rand() < mutation_rate ise:
        % O füzeye 1 ile num_targets arasında yeni rastgele bir hedef atayın (randi ile).
        
        % 4. YENİ BİREYİ KAYDET
        % Elde ettiğiniz bu son Çocuğu (Child), new_population'ın i. satırına kaydedin.
        
    end
    
    % --- POPÜLASYONU GÜNCELLEME ---
    % Artık eski popülasyon öldü. 
    % population matrisini new_population matrisine eşitleyin.
    
    % --- YENİ MALİYETLERİ HESAPLAMA VE EN İYİYİ BULMA ---
    % Yeni popülasyondaki tüm bireylerin cost değerlerini tekrar hesaplayın (fitness_function ile).
    % Yeni cost_array içindeki en küçük değeri (min) bulun.
    % Eğer bu yeni en küçük değer, bizim tarihi "best_cost"umuzdan daha iyiyse:
    % best_cost'u ve best_assignment'ı güncelleyin.
    
    % İterasyonun en iyi sonucunu kaydet
    convergence(iter) = best_cost;
    
end

end
```
