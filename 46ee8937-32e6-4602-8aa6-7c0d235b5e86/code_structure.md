# MATLAB Kod Yapısı: Weapon-Target Assignment

Makaledeki sistemi (Multi-to-Multi Interception, Grouping Constraints, ABC ve PSO algoritmaları) kodlamak için modüler bir MATLAB proje yapısı oluşturmanız en doğrusu olacaktır. Aşağıda adım adım oluşturmanız gereken `.m` dosyalarının taslaklarını (skeletons) ve birbirleriyle nasıl haberleşeceklerini bulabilirsiniz.

## Proje Dosya Dizini
Projenizde şu MATLAB dosyalarını oluşturmalısınız:
1. `main.m` (Ana çalıştırıcı dosya)
2. `calculate_probability.m` (Kesişim olasılıklarını hesaplayan matris fonksiyonu)
3. `fitness_function.m` (Atamaların başarımını ve ceza puanlarını hesaplayan fonksiyon)
4. `run_ABC.m` (Yapay Arı Kolonisi Algoritması)
5. `run_PSO.m` (Parçacık Sürüsü Optimizasyonu Algoritması)
6. `plot_results.m` (Grafik çizimleri)

---

## 1. `main.m`
Bu dosya senaryoyu tanımladığınız, algoritmaları çağırdığınız ve sonuçları görselleştirdiğiniz ana sürücüdür.

```matlab
% main.m
clear; clc; close all;

%% 1. Parametrelerin Tanımlanması (Case 1 & Case 2 için)
num_missiles = 8; % Örnek füze sayısı
num_targets = 4;  % Örnek hedef sayısı

% Makaledeki Case 1 verilerini buraya gireceksiniz:
% Hedef Tehdit Değerleri
target_threats = [0.8, 0.6, 0.9, 0.7]; 

% Kinematik parametrelerin tanımlanması (Pozisyon, hız vb.)
% ...

%% 2. Kesişim Olasılığı Matrisinin (P) Hesaplanması
% P(i,j) -> i. füzenin j. hedefi vurma olasılığı
P_matrix = calculate_probability(num_missiles, num_targets, /* kinematik_veriler */);

%% 3. Algoritma Parametreleri
max_iter = 100;
pop_size = 50;
grouping_type = 'adaptive'; % 'fixed' veya 'adaptive'
penalty_factor = 1000; % Adaptive grouping için ceza katsayısı

%% 4. ABC Algoritmasının Çalıştırılması
disp('ABC Algoritması Çalışıyor...');
[best_assignment_ABC, best_cost_ABC, convergence_ABC] = run_ABC(num_missiles, num_targets, P_matrix, target_threats, pop_size, max_iter, grouping_type, penalty_factor);

%% 5. PSO Algoritmasının Çalıştırılması
disp('PSO Algoritması Çalışıyor...');
[best_assignment_PSO, best_cost_PSO, convergence_PSO] = run_PSO(num_missiles, num_targets, P_matrix, target_threats, pop_size, max_iter, grouping_type, penalty_factor);

%% 6. Sonuçların Görselleştirilmesi ve Karşılaştırılması
plot_results(convergence_ABC, convergence_PSO, best_assignment_ABC, best_assignment_PSO);
```

---

## 2. `calculate_probability.m`
Bu fonksiyon, füzeler ve hedefler arasındaki uzaklık, görüş hattı oranı (LOS rate) ve zamanı (time-to-go) kullanarak her bir füze-hedef çifti için vurma ihtimalini hesaplar.

```matlab
function P = calculate_probability(num_missiles, num_targets, varargin)
    % P matrisi: satırlar füzeleri, sütunlar hedefleri temsil eder.
    P = zeros(num_missiles, num_targets);
    
    for i = 1:num_missiles
        for j = 1:num_targets
            % Makaledeki "Interception Probability Function" denklemlerini
            % buraya kodlayacaksınız.
            % miss_distance = ...
            % t_go = ...
            % los_rate = ...
            
            % Örnek sentetik hesaplama:
            % P(i,j) = hesaplanan_olasilik;
        end
    end
end
```

---

## 3. `fitness_function.m`
Hedeflerin vurulamama (hayatta kalma) ihtimalini minimize ederken, gruplama kısıtlarını (Grouping Constraints) kontrol eder.

```matlab
function [cost, is_valid] = fitness_function(assignment, P_matrix, target_threats, grouping_type, penalty_factor)
    % assignment: 1xN boyutunda bir vektör (N = füze sayısı)
    % Örnek: assignment = [1, 1, 2, 2, 3, 4] (1. ve 2. füze 1. hedefe, vb.)
    
    num_targets = length(target_threats);
    num_missiles = length(assignment);
    
    % 1. Temel Maliyet (Cost) Hesabı: Hedeflerin hayatta kalma tehdidi
    expected_survival = ones(1, num_targets);
    for i = 1:num_missiles
        target_idx = assignment(i);
        if target_idx > 0 && target_idx <= num_targets
            expected_survival(target_idx) = expected_survival(target_idx) * (1 - P_matrix(i, target_idx));
        end
    end
    
    % Toplam Tehdit
    total_threat = sum(target_threats .* expected_survival);
    cost = total_threat;
    is_valid = true;
    
    % 2. Gruplama Kısıtlarının Uygulanması
    penalty = 0;
    
    if strcmp(grouping_type, 'fixed')
        % Sabit Gruplama: Füzelerin hangi hedeflere kaçarlı atanacağı katı şekilde bellidir.
        % Eğer atama (assignment) bu kısıta uymuyorsa is_valid = false yapılır
        % veya sonsuz ceza verilir.
        % Örn: 1. ve 2. füze aynı hedefe gitmek ZORUNDA ise:
        if assignment(1) ~= assignment(2)
            is_valid = false;
            cost = inf;
        end
        
    elseif strcmp(grouping_type, 'adaptive')
        % Adaptif Gruplama: Makaledeki Ceza Fonksiyonu yöntemi.
        % Eğer istenilen gruplama yapısı bozulursa cost'a penalty eklenir.
        % Örn: Bir hedefe en fazla 2 füze atanabilir kısıtı
        for j = 1:num_targets
            num_assigned_to_j = sum(assignment == j);
            if num_assigned_to_j > 2 % Makaledeki formüle göre bu kısıt değişir
                penalty = penalty + penalty_factor * (num_assigned_to_j - 2);
            end
        end
        cost = cost + penalty;
    end
end
```

---

## 4. `run_ABC.m`
Makaledeki ABC (Artificial Bee Colony) algoritmasının WTA problemine uyarlanmış hali. Sürekli sayıları ayrık atamalara çevirmeniz gerekecektir.

```matlab
function [best_assignment, best_cost, convergence] = run_ABC(num_missiles, num_targets, P, threats, pop_size, max_iter, grouping_type, penalty_factor)
    % 1. İlklendirme (Initialization)
    population = zeros(pop_size, num_missiles);
    fitness = zeros(pop_size, 1);
    limit_count = zeros(pop_size, 1);
    limit_max = 20; % Kaşif arı limiti
    
    for i = 1:pop_size
        population(i, :) = randi([1, num_targets], 1, num_missiles); % Rastgele atama
        fitness(i) = fitness_function(population(i,:), P, threats, grouping_type, penalty_factor);
    end
    
    best_cost = min(fitness);
    best_assignment = population(find(fitness == best_cost, 1), :);
    convergence = zeros(max_iter, 1);
    
    % 2. Optimizasyon Döngüsü
    for iter = 1:max_iter
        % a) İşçi Arılar (Employed Bees)
        % Rastgele komşu bul ve daha iyiyse değiştir. (Örn: bir füzenin hedefini değiştir)
        
        % b) Gözcü Arılar (Onlooker Bees)
        % Rulet tekerleği ile iyi çözümleri seç ve etrafında araştırma yap.
        
        % c) Kaşif Arılar (Scout Bees)
        % limit_count(i) > limit_max ise o çözümü sil, rastgele yeni çözüm üret.
        
        % En iyi çözümü güncelle
        current_best = min(fitness);
        if current_best < best_cost
            best_cost = current_best;
            best_assignment = population(find(fitness == best_cost, 1), :);
        end
        convergence(iter) = best_cost;
    end
end
```

---

## 5. `run_PSO.m`
Makaledeki PSO karşılaştırması için. Standart PSO sürekli değerler üretir, bu nedenle pozisyonları hedef indislerine yuvarlamalısınız (round).

```matlab
function [best_assignment, best_cost, convergence] = run_PSO(num_missiles, num_targets, P, threats, pop_size, max_iter, grouping_type, penalty_factor)
    % 1. İlklendirme
    % Parçacıkların pozisyonları (1 ile num_targets arasında)
    positions = rand(pop_size, num_missiles) * (num_targets - 1) + 1;
    velocities = zeros(pop_size, num_missiles);
    
    pbest_positions = positions;
    pbest_costs = inf(pop_size, 1);
    gbest_position = zeros(1, num_missiles);
    gbest_cost = inf;
    
    convergence = zeros(max_iter, 1);
    
    w = 0.7; c1 = 1.5; c2 = 1.5; % PSO Parametreleri
    
    for iter = 1:max_iter
        for i = 1:pop_size
            % Sürekli pozisyonu tam sayıya (hedef indisine) çevir
            discrete_assignment = round(positions(i, :));
            discrete_assignment = max(1, min(num_targets, discrete_assignment)); % Sınırları koru
            
            % Maliyeti hesapla
            cost = fitness_function(discrete_assignment, P, threats, grouping_type, penalty_factor);
            
            % Pbest ve Gbest güncelle
            if cost < pbest_costs(i)
                pbest_costs(i) = cost;
                pbest_positions(i, :) = positions(i, :);
            end
            if cost < gbest_cost
                gbest_cost = cost;
                gbest_position = discrete_assignment;
            end
        end
        
        % Hız ve Pozisyon Güncellemesi
        r1 = rand(pop_size, num_missiles);
        r2 = rand(pop_size, num_missiles);
        velocities = w * velocities + c1 * r1 .* (pbest_positions - positions) + c2 * r2 .* (gbest_position - positions);
        positions = positions + velocities;
        
        % Sınırlandırma (1 ile num_targets arası)
        positions = max(1, min(num_targets, positions));
        
        best_assignment = gbest_position;
        best_cost = gbest_cost;
        convergence(iter) = best_cost;
    end
end
```

---

## 6. `plot_results.m`
Yakınsama eğrilerini ve matris sonuçlarını ekrana çizen fonksiyon.

```matlab
function plot_results(conv_ABC, conv_PSO, assign_ABC, assign_PSO)
    % Yakınsama Grafiği
    figure;
    plot(conv_ABC, 'b-', 'LineWidth', 2); hold on;
    plot(conv_PSO, 'r--', 'LineWidth', 2);
    xlabel('İterasyon (Iteration)');
    ylabel('Toplam Hedef Tehdidi (Total Threat)');
    legend('ABC Algoritması', 'PSO Algoritması');
    title('Algoritmaların Yakınsama Eğrisi Karşılaştırması');
    grid on;
    
    % Sonuçları Ekrana Yazdır
    disp('--- ABC Atama Sonuçları ---');
    disp(assign_ABC);
    disp('--- PSO Atama Sonuçları ---');
    disp(assign_PSO);
end
```
