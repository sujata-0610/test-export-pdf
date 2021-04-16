# test-export-pdf
test export pdf file
import { Component, OnDestroy, OnInit } from '@angular/core';
import { ChartDataSets, ChartOptions, ChartType } from 'chart.js';
import * as moment from 'moment';
import { Label } from 'ng2-charts';
import { AssetService } from '../services/data/asset.service';
import { FileService } from './../services/file/file.service';
import jsPDF from 'jspdf'

@Component({
  selector: 'app-alarm-summary',
  templateUrl: './alarm-summary.component.html',
  styleUrls: ['./maintenance.component.css']
})
export class AlarmSummaryReport implements OnInit, OnDestroy {

  constructor(private fileSvc: FileService) { 
  }
  fromDate: string;
  toDate: string;
  id : any;
  alarmarr: any[] = [];
  reportTitle : string;
  CompanyName : string;
  CompanyAddress : string;

  public barChartLabels: Label[] = [];
  public barChartType: ChartType = 'bar';
  public barChartPlugins = [];
  public barChartData: ChartDataSets[] = [];
  public barChartOptions: ChartOptions = {
    responsive: true
  };
  public headers = [];
  public data = [];

  ngOnInit(): void {
    this.id = 1;
    let date = new Date();
    date.setDate( date.getDate() - 7 );
    this.fromDate = moment(date).format('DD/MM/YYYY');
    date.setDate( date.getDate() + 6 );
    this.toDate = moment(date).format('DD/MM/YYYY');
    this.reportTitle = "Alarm Summary Report";
    this.CompanyName = " Neutron Manufacturing Pvt Ltd";
    this.CompanyAddress = "Sector - 26, Pradhikaran, Nigdi, Pune 411044";
    this.fileSvc.getAlarmSummaryHeaders().subscribe(
      data =>  {
        if (data != null && data.length > 0) {
          let headers = data.split('\n');
          headers = headers.filter(x => x.trim() !== '');
          for (const item of headers) {
            this.headers.push(item.trim());
          }
        } else {
          this.headers = [];
        }
      }
    );

    this.barChartLabels = [];
    this.barChartData = [];
    let dataValue : number[] = [];
    let OccurValue : number[] = [];
    this.fileSvc.getAlarmSummaryData().subscribe(
      data =>  {
        if (data != null && data.length > 0) {
          const tempData = data;
          let rows = [];
          rows = tempData.split('\n');
          let index = 0;
          for (let row of rows) {
            if (row.trim() === '') {
              continue;
            }
            row = row.replace('\r', '')
            const rowSplits = row.split(',');
            this.data[index++] = rowSplits;
            this.barChartLabels.push(rowSplits[1]); 
            dataValue.push(parseInt(rowSplits[4])); 
            OccurValue.push(parseInt(rowSplits[5])); 
          }
          this.barChartData.push({
            data:dataValue,
            label:"Total Time",
            backgroundColor: 'rgba(245,125,45,0.8)',
            borderColor: 'rgba(245,125,45,1)',
            pointBackgroundColor: 'rgba(245,125,45,0.5)',
            pointBorderColor: '#fff',
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: 'rgba(245,125,45,0.5)'
          });
          this.barChartData.push({
            data:OccurValue,
            label:"Occurrance",
            type:"line",
            backgroundColor: 'rgba(45,125,245,0.8)',
            borderColor: 'rgba(45,125,245,1)',
            pointBackgroundColor: 'rgba(45,125,245,0.5)',
            pointBorderColor: '#fff',
            pointHoverBackgroundColor: '#fff',
            pointHoverBorderColor: 'rgba(45,125,245,0.5)'
          });
          this.barChartData.reverse();
          this.barChartData.pop();
          console.log(this.barChartData);
        
        } else {
          this.data = [];
        }
      });
    
  }
  public selectedHeader = null;
  headerSeleced(header) {
    this.selectedHeader = header;
  }

  max: number = 200;
  showWarning: boolean;
  dynamic: number;
  type: string;

  stacked: any[] = [];

  timer: any = null;
  buttonCaption: string = 'Start';
  ngOnDestroy() {
    if (this.timer) {
      clearInterval(this.timer);
    }
  }
  downloadPdf() {

    var doc = new jsPDF({orientation : "portrait",
      unit: "pt",
      format: "a1"});
    
    doc.html(document.getElementById("printable"),{
      callback: function (doc) {
        doc.save("Alarm Summary.pdf");
      }
    });
  }

}
