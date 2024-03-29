# PrivacyPolicy68
Crazy Dots
**Privacy Policy**

Justtouch Games built the Crazy Dots app as a Free app. This SERVICE is provided by Justtouch Games at no cost and is intended for use as is.

This page is used to inform visitors regarding my policies with the collection, use, and disclosure of Personal Information if anyone decided to use my Service.

If you choose to use my Service, then you agree to the collection and use of information in relation to this policy. The Personal Information that I collect is used for providing and improving the Service. I will not use or share your information with anyone except as described in this Privacy Policy.

The terms used in this Privacy Policy have the same meanings as in our Terms and Conditions, which are accessible at Crazy Dots unless otherwise defined in this Privacy Policy.

**Information Collection and Use**

For a better experience, while using our Service, I may require you to provide us with certain personally identifiable information. The information that I request will be retained on your device and is not collected by me in any way.

The app does use third-party services that may collect information used to identify you.

Link to the privacy policy of third-party service providers used by the app

*   [Google Play Services](https://www.google.com/policies/privacy/)
*   [AdMob](https://support.google.com/admob/answer/6128543?hl=en)
*   [Google Analytics for Firebase](https://firebase.google.com/policies/analytics)
*   [Firebase Crashlytics](https://firebase.google.com/support/privacy/)
*   [Unity](https://unity3d.com/legal/privacy-policy)

**Log Data**

I want to inform you that whenever you use my Service, in a case of an error in the app I collect data and information (through third-party products) on your phone called Log Data. This Log Data may include information such as your device Internet Protocol (“IP”) address, device name, operating system version, the configuration of the app when utilizing my Service, the time and date of your use of the Service, and other statistics.

**Cookies**

Cookies are files with a small amount of data that are commonly used as anonymous unique identifiers. These are sent to your browser from the websites that you visit and are stored on your device's internal memory.

This Service does not use these “cookies” explicitly. However, the app may use third-party code and libraries that use “cookies” to collect information and improve their services. You have the option to either accept or refuse these cookies and know when a cookie is being sent to your device. If you choose to refuse our cookies, you may not be able to use some portions of this Service.

**Service Providers**

I may employ third-party companies and individuals due to the following reasons:

*   To facilitate our Service;
*   To provide the Service on our behalf;
*   To perform Service-related services; or
*   To assist us in analyzing how our Service is used.

I want to inform users of this Service that these third parties have access to their Personal Information. The reason is to perform the tasks assigned to them on our behalf. However, they are obligated not to disclose or use the information for any other purpose.

**Security**

I value your trust in providing us your Personal Information, thus we are striving to use commercially acceptable means of protecting it. But remember that no method of transmission over the internet, or method of electronic storage is 100% secure and reliable, and I cannot guarantee its absolute security.

**Links to Other Sites**

This Service may contain links to other sites. If you click on a third-party link, you will be directed to that site. Note that these external sites are not operated by me. Therefore, I strongly advise you to review the Privacy Policy of these websites. I have no control over and assume no responsibility for the content, privacy policies, or practices of any third-party sites or services.

**Children’s Privacy**

These Services do not address anyone under the age of 13. I do not knowingly collect personally identifiable information from children under 13 years of age. In the case I discover that a child under 13 has provided me with personal information, I immediately delete this from our servers. If you are a parent or guardian and you are aware that your child has provided us with personal information, please contact me so that I will be able to do the necessary actions.

**Changes to This Privacy Policy**

I may update our Privacy Policy from time to time. Thus, you are advised to review this page periodically for any changes. I will notify you of any changes by posting the new Privacy Policy on this page.

This policy is effective as of 2022-02-19

**Contact Us**

If you have any questions or suggestions about my Privacy Policy, do not hesitate to contact me at justtouchinfo@gmail.com.


1

string kaynakConnectionString = "Data Source=KAYNAK_VERITABANI.db";
string hedefConnectionString = "Data Source=HEDEF_VERITABANI.db";

using (var kaynakConnection = new SQLiteConnection(kaynakConnectionString))
using (var hedefConnection = new SQLiteConnection(hedefConnectionString))
{
    kaynakConnection.Open();
    hedefConnection.Open();
    
    // "rescuver" adlı kolonun 0 basılması ve diğer tabloda olmayan kolonların null basılması için SQL sorgusu
    string transferSorgusu = @"
        INSERT INTO HedefTablo (ID, Isim, rescuver, DiğerKolon)
        SELECT K.ID, K.Isim, 0 AS rescuver, NULL AS DiğerKolon
        FROM KaynakTablo K
        LEFT JOIN HedefTablo H ON K.ID = H.ID
        WHERE H.ID IS NULL
    ";

    using (var transferKomut = new SQLiteCommand(transferSorgusu, kaynakConnection))
    {
        transferKomut.ExecuteNonQuery();
    }
}
2


string kaynakConnectionString = "Data Source=KAYNAK_VERITABANI.db";
string hedefConnectionString = "Data Source=HEDEF_VERITABANI.db";

// DataContext oluşturma
using (var kaynakDataContext = new DataContext(kaynakConnectionString))
using (var hedefDataContext = new DataContext(hedefConnectionString))
{
    // Kaynak ve hedef tabloların IQueryable nesnelerini alın
    var kaynakTablo = kaynakDataContext.GetTable<KaynakTablo>();
    var hedefTablo = hedefDataContext.GetTable<HedefTablo>();

    // LINQ sorgusu kullanarak veri transferi
    var eksikSatirlar = from kaynak in kaynakTablo
                        join hedef in hedefTablo on kaynak.ID equals hedef.ID into gj
                        from subhedef in gj.DefaultIfEmpty()
                        where subhedef == null
                        select new HedefTablo
                        {
                            ID = kaynak.ID,
                            Isim = kaynak.Isim,
                            rescuver = 0, // rescuver kolonunu 0 olarak atama
                            DiğerKolon = null // DiğerKolon kolonunu null olarak atama
                        };

    // Eksik satırları hedef tabloya ekleme
    hedefTablo.InsertAllOnSubmit(eksikSatirlar);
    hedefDataContext.SubmitChanges();
}


3

using System;
using System.Data.SQLite;

namespace VeriBirlestirmeUygulamasi
{
    class Program
    {
        static void Main(string[] args)
        {
            string kaynakBaglantiString = "Data Source=kaynakVeritabani.db;Version=3;";
            string hedefBaglantiString = "Data Source=hedefVeritabani.db;Version=3;";

            using (SQLiteConnection kaynakBaglanti = new SQLiteConnection(kaynakBaglantiString))
            using (SQLiteConnection hedefBaglanti = new SQLiteConnection(hedefBaglantiString))
            {
                kaynakBaglanti.Open();
                hedefBaglanti.Open();

                string kaynakSorgu = "SELECT id, ad, soyad, yaş FROM KaynakTablo";
                SQLiteCommand kaynakKomut = new SQLiteCommand(kaynakSorgu, kaynakBaglanti);
                SQLiteDataReader kaynakOkuyucu = kaynakKomut.ExecuteReader();

                while (kaynakOkuyucu.Read())
                {
                    int id = kaynakOkuyucu.GetInt32(0);
                    string ad = kaynakOkuyucu.GetString(1);
                    string soyad = kaynakOkuyucu.GetString(2);
                    int yas = kaynakOkuyucu.GetInt32(3);

                    // Hedef tabloda aynı satırın tamamen aynısı var mı kontrol et
                    string hedefKontrolSorgu = "SELECT COUNT(*) FROM HedefTablo WHERE id = @id AND ad = @ad AND soyad = @soyad AND yaş = @yas";
                    SQLiteCommand hedefKontrolKomut = new SQLiteCommand(hedefKontrolSorgu, hedefBaglanti);
                    hedefKontrolKomut.Parameters.AddWithValue("@id", id);
                    hedefKontrolKomut.Parameters.AddWithValue("@ad", ad);
                    hedefKontrolKomut.Parameters.AddWithValue("@soyad", soyad);
                    hedefKontrolKomut.Parameters.AddWithValue("@yas", yas);

                    int satirSayisi = Convert.ToInt32(hedefKontrolKomut.ExecuteScalar());

                    if (satirSayisi == 0)
                    {
                        // Hedef tabloda aynı satırın tamamen aynısı yoksa ekle
                        string hedefEklemeSorgu = "INSERT INTO HedefTablo (id, ad, soyad, yaş, cinsiyet, kilo, boy, hobiler) " +
                                                   "VALUES (@id, @ad, @soyad, @yas, NULL, 0, 0, NULL)";

                        SQLiteCommand hedefEklemeKomut = new SQLiteCommand(hedefEklemeSorgu, hedefBaglanti);
                        hedefEklemeKomut.Parameters.AddWithValue("@id", id);
                        hedefEklemeKomut.Parameters.AddWithValue("@ad", ad);
                        hedefEklemeKomut.Parameters.AddWithValue("@soyad", soyad);
                        hedefEklemeKomut.Parameters.AddWithValue("@yas", yas);

                        hedefEklemeKomut.ExecuteNonQuery();
                    }
                    // Eğer aynı satırın tamamen aynısı varsa atla ve bir sonraki kayda geç
                }

                kaynakBaglanti.Close();
                hedefBaglanti.Close();
            }
        }
    }
}
