import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Calendar;


public class SharePrice {
	public static final String TIMESTAMP_FORMAT = "MMM-yyyy";
	private static Calendar calendar = Calendar.getInstance();
	private ThreadLocal<DateFormat> dtFormat = new ThreadLocal<DateFormat>() {
		@Override
		protected DateFormat initialValue() {
			return new SimpleDateFormat(TIMESTAMP_FORMAT); 
		}
	};
	/**
	 * @param args
	 */
	public static void main(String[] args) {		
		SharePrice sp = new SharePrice();
		try {
			
			sp.getData();
			
		} catch (IOException e) {
			System.out.println("SharePrice.main: CSV not found");
		}	 
	}

	public void getData() throws IOException {
		BufferedReader br = null;
		String line = ""; 
		String cvsSplitBy = ",";

		try {
			String csvFile = "sharePrice.csv";
			File file = new File(csvFile);
			br = new BufferedReader(new InputStreamReader(new FileInputStream(file)));
			String headerData[] = br.readLine().split(cvsSplitBy); // ignore first row
			int index[]= new int[headerData.length-2];		//Month,year			
			int hPrice[]= new int[headerData.length-2]; 	//highest price			
			int internalIndex = 1;
			
			while ((line = br.readLine()) != null) {      
				String[] csvData = line.split(cvsSplitBy);

				for (int i = 2; i < csvData.length; i++) {
					int val = Integer.valueOf(csvData[i]);
					if(val > hPrice[i-2]) {
						hPrice[i-2] = val;
						index[i-2] = internalIndex;
					}
				}
				internalIndex++;
			}	 
			for (int i = 0; i < hPrice.length; i++) 
				System.out.println(headerData[i+2] + "\t" + hPrice[i] + "\t" + getMonthAndYear(index[i]));	//company names read from headerData 
		} finally {
			if (br != null) {
				try {
					br.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}

	private String getMonthAndYear(int i) {
		calendar.clear();
		calendar.set(1990, 0, 1);		//should be read from headerData
		calendar.add(Calendar.MONTH, i-1);
		String date = dtFormat.get().format(calendar.getTime());
		//		System.out.println("SharePrice.main " + date);
		return date;
	}
}
